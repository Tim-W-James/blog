Creating forms that elegantly handle validation in an accessible, maintainable and extendable way is a challenge. Using `zod` and `react-hook-form` we can take a declarative approach that decouples our form, validation and component logic.

We'll be using the following libraries:

- [zod](https://zod.dev/): Runtime validation allowing us to declare schemas that parse data users input and generates a validation message.
- [react-hook-form](https://react-hook-form.com/): Gives us a hook to control our form and trigger validation.
- [Material UI](https://material-ui.com/): Component library to style our form input fields.

It should be noted that apart from `react-hook-form`, these libraries are interchangeable. For example, you could use `io-ts` instead of `zod` to define your form schemas, or Bootstrap instead of Material UI.

## Rendering a Simple Field

Let's start by creating a simple uncontrolled field for the user to input their email.

```tsx
<form noValidate>
  <TextField
    name="email"
    label="Email"
    required
    placeholder="example@mail.com"
  />
</form>
```

A few things to note:

- `label="Email"` defines the visual label on the MUI (Material UI) component, while `name="email"` is the native HTML `name` attribute which allows the browser to make suggestions among other things (so you can quickly enter the email you've used on other sites). This may not be desirable in some cases, so you can choose to omit the `name` prop or add `autoComplete="off"` to the `form` element.
- We use `noValidate` on the `form` to disable native HTML validation. We'll be using our own custom `zod` codec to define our own flexible validation rules to and have a consistent UX across different browsers.
- We haven't specified an `onChange` or `value` prop, so the field is currently uncontrolled. We'll be using `react-hook-form` to manage this for us.
- `required` adds an HTML `required` attribute which has accessibility properties, and adds a `*` to the label.

## Controlling the Form

Let's introduce `react-hook-form` to control this input. The core of this library is the `useForm` hook, lets add that to our form:

```tsx
const { control, handleSubmit } = useForm({
  defaultValues: {
    email: "",
  },
});
```

At the moment, we only have a single field so we're just saying that our email input field is empty by default. You'll also notice we don't have anything describing the 'shape' of our form, or any type safety for the data our form contains - keep that in mind for later.

We now have a form `control`, which allows us to do a lot of magic like watch the data a user inputs into a field, handle validation, and much more. But we need to tell this `control` what the `email` input component is. To do this, we wrap the `TextField` in a `Controller` like so:

```tsx
<Controller
  name="email"
  control={control}
  render={({
    field: { value, onChange, onBlur, ref },
    fieldState: { error },
  }) => (
    <TextField
      name="email"
      label="Email"
      placeholder="example@mail.com"
      required
      inputRef={ref}
      value={value}
      onChange={onChange}
      onBlur={onBlur}
      error={Boolean(error)}
    />
  )}
/>
```

This does a few things:

- `name="email"` on the `Controller` tells our form controller (`control={control}`) which field this is. Again, this is different to the `name="email"` on `TextField` which is used by the browser for suggestions, etc.
- `render` passes a few props into the MUI `TextField`:
  - `value` and `onChange` so that the input can be managed by `react-hook-form`.
  - `ref` so that the controller can move the user's focus to the input field (e.g., when trying to submit the form and the field is in an invalid state).
  - `error` just gives the MUI component a red outline to make it visually clear when the field is invalid.
  - `onBlur` so that the controller can trigger events (such as validation) when the user is no longer focusing the input.

Note that `react-hook-form` can also [register](https://react-hook-form.com/get-started#Registerfields) fields instead. Since we're using a component library, we need to use a `Controller`, but `register` will work if you're using a custom HTML `input`.

Let's also add some logic to submit our form:

- A button within `form`:

  ```tsx
  <Button type="submit">Submit</Button>
  ```

- A function to run on submit (this could be submitting form data to an API,
  etc.):

  ```tsx
  const onSubmit = (data) => console.log(data);
  ```

- Then hook it up to our `form` with the prop:

  ```tsx
  onSubmit={handleSubmit(onSubmit)}
  ```

You should now be able to hit the button and see a JSON object with your inputted email logged to the console. But we have a few problems:

- The email is required, but we can still submit our form without specifying it.
- We can enter an email in an invalid format, and our form will still accept it.
- We don't have any way to display messages to the user when these validation rules fail.
- As mentioned earlier, we still don't have good type safety.

## Creating a Schema

First, let's make the email field required. To do this, we're going create a `zod` schema. While this is a relatively simple example, a library like `zod` allows us to:

- Define a set of validation rules and a dynamic message we can display to the user depending on any violated rule.
- Allows us to infer TypeScript definitions from that schema.
- `transform` or `coerce` data inputted by the user. Maybe we need to convert an `unknown` something to a `number`, or strip whitespace. Importantly, this 'parsing' logic is tied to the validation logic and happens at the boundaries of our application.
- More generally, gives us runtime type checking, so we can make sure the raw data inputted by the user is in a form that our application can consume (why is this important? Read [this](https://medium.com/agiledigital/creating-custom-io-ts-decoders-for-runtime-parsing-ffbfc8705bfa)).

We're going to start with:

```tsx
const formSchema = z.object({
  email: z.string().nonempty(),
});
type FormFields = z.infer<typeof formSchema>;
```

then pass it to the `useForm` hook using the `zodResolver` from [`@hookform/resolvers`](https://www.npmjs.com/package/@hookform/resolvers):

```tsx
const { control, handleSubmit } = useForm<FormFields>({
  mode: "onBlur",
  reValidateMode: "onBlur",
  resolver: zodResolver(formSchema),
  defaultValues: {
    email: "",
  },
});
```

Boom. You should be able to hit submit without specifying an email, then the outline will turn red and be focused. We don't disable the submit button if the form is in an invalid state. This allows the user to more easily identify what in the form is invalid (you can imagine how important this is in a much larger form than this). You may want to render the button differently, but it should still be interactive. Let's break down what's happening here:

- We define that the `email` field is a `string()` that is `nonempty()` using `zod`. Fields in `zod` are `required` by default. The nice thing about `zod` is that by default it strips any unexpected keys that aren't defined in schema, which makes your application more secure ([read more](https://cheatsheetseries.owasp.org/cheatsheets/Mass_Assignment_Cheat_Sheet.html)).
- We `infer` the TypeScript type `FormFields` and pass it to the `useForm` type param, so now the output `data` is properly typed and any `Controller` we define.
- Set the validation `mode` to `onBlur`. This is ideal for accessibility, since the user validation status doesn't constantly change as the user types, but they don't need to submit before knowing whether that field is valid or not. We also set `reValidateMode` so that the behaviour is consistent after the
  first submit.

It's worth noting that `react-hook-form` has it's own built-in [rules](https://react-hook-form.com/get-started#Applyvalidation), but these don't give us anywhere near the flexibility of `zod`.

## Displaying a Validation Error

Next, let's improve this by rendering out the validation message to the user. Update the `render` prop within the `Controller`:

```tsx
render={({
  field: { value, onChange, onBlur, ref },
  fieldState: { error },
}) => (
  <FormControl>
    <TextField
      name="email"
      label="Email"
      placeholder="example@mail.com"
      required
      inputRef={ref}
      value={value}
      onChange={onChange}
      onBlur={onBlur}
      error={Boolean(error)}
    />
    <FormHelperText
      sx={{
        color: 'error.main',
      }}
    >
      {error?.message ?? ''}
    </FormHelperText>
  </FormControl>
)}
```

We're using the MUI `FormControl` and `FormHelperText` to associate an error message with the input field, if one exists.

Now we can see an error, but not one that is particularly readable:

> String must contain at least 1 character(s)

That's because we're using the built in `zod` error reporter. We can define our
own message instead:

```tsx
const formSchema = z.object({
  email: z.string().nonempty("Please specify an email"),
});
```

However, we still allow an invalid email. Thankfully, `zod` has a built in
`email()` schema:

```tsx
const formSchema = z.object({
  email: z
    .string()
    .nonempty("Please specify an email")
    .email("Please specify a valid email"),
});
```

Now we have a dynamic error message to help the user identify what they got wrong.

## Adding Another Field

Let's extend our form with an optional mobile phone number. This has some implications to consider:

- We need to make sure the input only contains digits. But a user might try to enter the number in different formats (`0400-000-000` or `0400 000 000`).
- It must be exactly 10 digits long.
- It must start with "04".

Let's translate that into schema:

```tsx
const mobilePhoneNumberSchema = z
  .string()
  // Remove all non-digit characters.
  .transform((value) => value.replace(/\D/gu, ""))
  // Must be 10 digits.
  .refine((value) => value.length === 10, "Please specify 10 digits")
  // Mobile, Cellular, and Satellite services use the code 04.
  .refine(
    (value) => value.startsWith("04"),
    "Mobile numbers must start with '04'"
  );
```

There are two new parts to this schema:

- `transform` allows us to manipulate the raw value inputted by the user.
- Instead of using a built-in validation rule like `email`, we define our own custom checks with `refine`.

Now we need to add it to the form schema:

```tsx
const formSchema = z.object({
  phone: mobilePhoneNumberSchema,
  ...
```

set the default in the `useForm` hook:

```tsx
    defaultValues: {
      phone: '',
      ...
```

and add the input field within the `form`:

```tsx
<form noValidate onSubmit={handleSubmit(onSubmit)}>
  <Controller
    name="phone"
    control={control}
    render={({
      field: { value, onChange, onBlur, ref },
      fieldState: { error },
    }) => (
      <FormControl>
        <TextField
          name="phone"
          label="Phone"
          placeholder="0400-000-000"
          inputRef={ref}
          value={value}
          onChange={onChange}
          onBlur={onBlur}
          error={Boolean(error)}
        />
        <FormHelperText
          sx={{
            color: 'error.main',
          }}
        >
          {error?.message ?? ''}
        </FormHelperText>
      </FormControl>
    )}
  />
  ...
```

One last thing - we want to make the field optional. We could update some logic in our reusable `mobilePhoneNumberSchema`, but we might want to use that in other parts of our app where it is required. Instead, let's try adding `.optional()` to `mobilePhoneNumberSchema` in the `formSchema`.

Looks good right? Well not quite. What happens if we enter something, then delete it? It still says:

> Please specify 10 digits

So we aren't quite there. This is because `.optional()` actually only checks if the value is `undefined`, and once the field is dirty, the value instead becomes an empty string. There are a few other cases where this can happen, such as an MUI select component being `null`. Here is a helper that will `preprocess` (similar to `transform`, but happens before parsing) empty strings and `null`s to `undefined` to make the optional behaviour work as expected:

```tsx
const looseOptional = <T extends z.ZodTypeAny>(schema: T) =>
  z.preprocess(
    (value: unknown) =>
      value === null || (typeof value === "string" && value === "")
        ? undefined
        : value,
    schema.optional()
  );
```

Now we can wrap the `mobilePhoneNumberSchema` with that utility and get the behaviour we expect:

```tsx
const formSchema = z.object({
  phone: looseOptional(mobilePhoneNumberSchema),
  ...
```

## Some Extra Gotchas

- Make sure you only specify `required` on the input component when it is also required in schema.
- You may have multiple forms or even nested forms in complex scenarios, in which case a button with `type="submit"` might get in your way and trigger submit on multiple forms (very annoying). This is because native HTML is once again getting in our way, you can override it by changing the `form` `onSubmit`:

  ```tsx
  onSubmit={(event) => {
    // Prevent the event from triggering other active forms.
    event.preventDefault();
    event.stopPropagation();

    void handleSubmit(onSubmit)(event);
  }}
  ```

- If you need to coerce a value (e.g., a `Date` via [MUI date picker](https://mui.com/x/react-date-pickers/getting-started/)), you can use `z.coerce.date()` in addition to [other primitives](https://zod.dev/?id=coercion-for-primitives) respectively. However, you may want a custom validation message in some cases, in which case you'll need to use an error map instead:

  ```tsx
  z.date({
    errorMap: (issue, ctx) =>
      issue.code === z.ZodIssueCode.invalid_date
        ? { message: "Please specify a date" }
        : { message: ctx.defaultError },
    coerce: true,
  });
  ```

## Wrapping Up

Using these three libraries, we are able to decouple our form, validation, and component logic, meaning we can easily define reusable schemas, or swap to a completely different component library with ease. We also get the safety of runtime type checking, while having accessible user-friendly forms with helpful validation messages.

You can find the source code for the example on [Stackblitz](https://stackblitz.com/edit/stackblitz-starters-1f3swh?file=src%2FApp.tsx).

## Further Reading

- [Creating Custom io-ts Decoders for Runtime Parsing](https://medium.com/agiledigital/creating-custom-io-ts-decoders-for-runtime-parsing-ffbfc8705bfa)
- [Parse, don’t validate](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/)
- Stay tuned for a follow up post on implementing conditional fields.
