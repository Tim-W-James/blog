Applying classes directly to a React component provides no type safety by default. This can lead to errors, unexpectedly missing styles, and a lesser development experience without intellisense. In this blog, I'll be achieving type safety for both [Tailwind](https://tailwindcss.com/) and [SCSS (Sass) Modules](https://sass-lang.com/).

## The Development Experience

You can find the minimal repo with this config [here](https://github.com/Tim-W-James/typed-tailwind-and-scss-modules).

### Tailwind CSS

Instead of using a class name directly, this works by using a custom utility function. For example:

```tsx
import cn from "../styles/cssUtils";
...
<div className={cn("container p-5", { "text-lg": true })}>
```

This works by generating CSS class names for any Tailwind or global classes from the compiled CSS, and generating a union type in [`cssClasses.d.ts`](./src/styles/cssClasses.d.ts). This is automatically generated during development (`npm run dev`). Note that unused Tailwind classes are purged and excluded from the type definition, so when adding a new class your IDE will complain momentarily until the file is saved and types are re-generated.

### SCSS Modules

To use scoped **SCSS modules** with type safety, separate `./...module.scss.d.ts` files are generated in the directory of it's parent component. This can be used with the `cnScoped` function, for example:

```tsx
import { cnScoped } from "../styles/cssUtils";
import styles from "./component.module.scss";
...
<div className={cnScoped(styles)(styles._component, "container p-5", {
  "text-lg": true,
})}>
...
```

Some things to note:

- Classes in SCSS modules are named with a `_` prefix and are lowerCamelCase.
- Using the `styles` import is required for the module to be compiled and avoid
  name collisions, and gives intellisense.
- The `ClassNames` type provides `cnScoped` with a union of all class names in
  that SCSS module, so that it knows what valid classes are in scope. Note the
  extra `()` since the function needs to by curried.

## How it Works

Let's go into detail on how to set things up and how the magic happens.

### Generating Types

First we need to generate types. For Tailwind, we can do this using [`postcss-ts-classnames`](https://github.com/esamattis/postcss-ts-classnames), which essentially creates a big union type of all the classes it finds in the bundled app. Importantly, this happens **after** unused classes are purged, so we don't get the entire list of possible Tailwind classes. This is ideal for performance reasons, but does mean there is a small lag between a class being used in code and the generated types being updated.

Update your `vite.config.ts` to include this config (`postCssTsClassnames` is the interesting part):

```ts
import postCssTsClassnames from 'postcss-ts-classnames';

...

css: {
  postcss: {
    plugins: [
      tailwind,
      autoprefixer,
      postCssTsClassnames({
        dest: 'src/styles/cssClasses.d.ts',
        // Set isModule if you want to import ClassNames from another file
        isModule: true,
        exportAsDefault: true, // to use in combination with isModule
      }),
    ],
  },
},
```

Given the classes:

```tsx
<div className="container p-5 text-teal-300">
```

It generates the file `styles/cssClasses.d.ts`:

```ts
// This file is auto-generated with postcss-ts-classnames.

export type ClassNames = "container" | "p-5" | "text-teal-300";

export default ClassNames;
```

To generate types for SCSS modules, we use [`vite-plugin-sass-dts`](https://github.com/activeguild/vite-plugin-sass-dts). We simply need to add this to our list of Vite plugins:

```ts
plugins: [react(), sassDts()],
```

Given a SCSS module `MyTypeSafeComponent.module.scss`:

```scss
._myClass {
  color: red;
}
```

It generates the file `MyTypeSafeComponent.module.scss.d.ts`:

```ts
declare const classNames: {
  readonly _myClass: "_myClass";
};
export = classNames;
```

One more thing - we need to ignore the auto generated files to stop Vite getting stuck in a cycle of generating the types, noticing that the files storing these types has changes, then generating the types again, and again... We can do this by ignoring these patterns in the `vite.config.ts`:

```ts
server: {
  watch: {
    ignored: ['**/cssClasses.d.ts', '**/*module.scss.d.ts'],
  },
},
```

### Using the Types

To use the global Tailwind types from `styles/cssClasses.d.ts`, I've leveraged a lot of work from [this post](https://kimmo.blog/posts/6-advanced-typescript-the-ultimate-tailwind-typings/), so credit goes there for a lot of the complex TypeScript wizardry that makes things work. In essence, it builds upon the [`classnames`](https://www.npmjs.com/package/classnames) (or [`clsx`](https://www.npmjs.com/package/clsx)) to provide a helper function that gives us with the type safety we're after. This cleverness means we get type checking that works with whitespace, multiple classes (e.g., `"container p-5"`)and arbitrary values (e.g., `"border-[5px]"`). The input `"container p-5 invalid-class"` provides the nifty error message:

> 'invalid-class' is not a valid Tailwind or scoped class

The end result is we get this:

```tsx
<div
  className={cn(
    // Multiple classes can be a separate param, or within the same string
    "container p-5",
    // Can also use a condition
    { "text-teal-300": true }
  )}
>
  Type Safe Tailwind!
</div>
```

I've adapted Kimmo's code to work with scoped SCSS modules, all within the same function. Importantly, our Tailwind classes file will be used globally, while the SCSS modules need to be scoped to specific components. There are a few steps to make this happen:

- We need to exclude the SCSS classes from the global set of types. We only want to allow classes that are explicitly within scope of a SCSS module. My solution to this is to prefix these SCSS classes with an underscore `_`, so they can be differentiated from Tailwind types. Optionally, you can also enforce this convention with this Stylelint rule:

  ```ts
  "selector-class-pattern": [
    "^_[a-z][a-zA-Z0-9]+$",
    {
      message:
        "Expected class name to be prefixed with '_' and be lowerCamelCase",
    },
  ]
  ```

  Which we can then override for the `styles` directory so we can have some global SCSS too `styles/.stylelintrc.cjs`:

  ```ts
  module.exports = {
    extends: "../../.stylelintrc.cjs",
    rules: {
      "selector-class-pattern": [
        "^([a-z][a-z0-9]*)(-[a-z0-9]+)*$",
        {
          message: "Expected class name to be kebab-case",
        },
      ],
    },
  };
  ```

- We need a way to specify the generated types from our SCSS module. Unfortunately (at least to my understanding...), TypeScript can't mix inferred and explicit types, so we need to curry the function, then we can allow those types.

In action, this looks like:

```tsx
import { cnScoped } from "../styles/cssUtils";
import styles from "./MyTypeSafeComponent.module.scss";

const MyTypeSafeComponent: React.FC = () => (
  <div
    className={cnScoped(styles)(
      styles._myClass,
      // Can be mixed with Tailwind classes
      "container p-5"
    )}
  >
    Type Safe SCSS Modules!
  </div>
);
```

`styles` has intellisense and is fully scoped to that component. TypeScript won't let us use an invalid class, a class from an out-of-scope SCSS module, and will work if two classes in different SCSS modules have a name collision. 🚀

## Gotchas

- Due to a limitation with how TypeScript infers template literals, if a class exists which is a prefix, it will break other classes that use that prefix. For example: `flex-col` will be marked as invalid because `flex` is a class. As a workaround, you can pass these classes as separate parameters:

  ```ts
  cn("flex p-5", "flex-col", "flex-wrap");
  ```

- There are some cases when the Vite dev server is running, but you don't want to regenerate types. One example is when using Storybook. A workaround is to use an environment variable to disable the type generation:

  ```tsx
  const enableCssTypeGen = !(process.env.DISABLE_CSS_TYPE_GEN === "true");
  ```

## Further Reading / Resources

- Kimmo's blog on typed Tailwind: <https://kimmo.blog/posts/6-advanced-typescript-the-ultimate-tailwind-typings/>
- The source code for my website uses this extensively: <https://github.com/Tim-W-James/timjames.dev>
- Minimal repo with this configured: <https://github.com/Tim-W-James/typed-tailwind-and-scss-modules>
