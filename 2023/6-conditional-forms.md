When building forms, there are complex cases where the validation of one field may depend on the state of another. In this post, I'll implement an example of one such scenario using [`zod`](https://zod.dev/), [`react-hook-form`](https://react-hook-form.com/) and [Material UI](https://material-ui.com/).

Consider the case where a user is providing contact information, and they have several options (email or phone number). Then they need to pick their preferred contact method. We want to validate that the user has entered the contact details for the preferred method they chose, and have other contact details be optional. We need a way to check the validity of the email or phone fields conditional on the chosen preferred contact method.

This post is the second in a series on building forms with React, you can read the previous post [here](https://dev.to/timwjames/building-forms-with-zod-and-react-hook-form-2geg).

## Creating the Autocomplete Field

First, let's create the input field component using Material UI. This part will be dealing with integrating an MUI component with `react-hook-form`, so if you're less interested that and want to skip to the dependent validation logic, jump to <a href="#updating-the-schema-with-dependent-fields"># Updating the Schema with Dependent Fields</a>.

To do this, we'll be using [Autocomplete](https://mui.com/material-ui/react-autocomplete/), which will give us a dropdown with a list of items we can pick. It also has some extra built-in functionality over a basic [select](https://mui.com/material-ui/react-select/) element, that gives us the ability to type and filter for items in the list, and clear the input once selected.

At it's most basic, we can use the component like this:

```tsx
<Autocomplete
  options={["Email", "Phone"]}
  sx={{ width: 245 }}
  renderInput={(params) => (
    <TextField {...params} required label={"Preferred Contact Method"} />
  )}
/>
```

`Autocomplete` acts as wrapper around the base `TextField`, where we specify the set of possible `options`. If you've read the previous [tutorial](https://dev.to/timwjames/building-forms-with-zod-and-react-hook-form-2geg), you'll know what's coming next; we need to control the input using `react-hook-form` and update our `zod` schema for validation.

Pull out the set of options into a constant, since we'll be using it in a few places:

```tsx
const contactMethods = ["Email", "Phone"] as const;
```

Update the `zod` schema (you can find the code from the previous tutorial on [Stackblitz](https://stackblitz.com/edit/stackblitz-starters-1f3swh?file=src%2FApp.tsx)) to include our new field, using a [zod enum](https://zod.dev/?id=zod-enums):

```tsx
const formSchema = z.object({
  phone: looseOptional(mobilePhoneNumberSchema),
  email: z
    .string()
    .nonempty("Please specify an email")
    .email("Please specify a valid email"),
  preferredContactMethod: z.enum(contactMethods),
});
```

Finally, add the `Autocomplete` component to a new `react-hook-from` `Controller` within the `form` (again, referring to [code from the previous tutorial](https://stackblitz.com/edit/stackblitz-starters-1f3swh?file=src%2FApp.tsx)):

```tsx
<Controller
  name="preferredContactMethod"
  control={control}
  render={({
    field: { value, onChange, onBlur, ref },
    fieldState: { error },
  }) => (
    <FormControl>
      <Autocomplete
        onChange={(
          _event: unknown,
          item: (typeof contactMethods)[number] | null
        ) => {
          onChange(item);
        }}
        value={value ?? null}
        options={contactMethods}
        sx={{ width: 245 }}
        renderInput={(params) => (
          <TextField
            {...params}
            required
            error={Boolean(error)}
            onBlur={onBlur}
            inputRef={ref}
            label={"Preferred Contact Method"}
          />
        )}
      />
      <FormHelperText
        sx={{
          color: "error.main",
        }}
      >
        {error?.message ?? ""}
      </FormHelperText>
    </FormControl>
  )}
/>
```

There are a few nuances here, let's break things down:

- We're passing the `value` and `onChange` to the `Autocomplete` component. We can't use the `onChange` directly. We don't care about the `event` of the callback, only the `item`, and unfortunately MUI doesn't infer the types for us. Using TypeScript, we extract the union type from our array of `contactMethods` via `typeof contactMethods[number]`, but we also need to include `null` (more on that later). Then we can pass that into the `onChange` from `react-hook-form`'s `Controller` in the callback.
- A few props `react-hook-form` uses to control the form need to be passed to the `TextField` itself, namely: `error`, `onBlur` and `ref`. This allows `react-hook-form` to put the field in an error state, trigger revalidation on blur, and focus the input respectively.
- Like other fields, we use MUI's `FormControl` and `FormHelperText` to render the error message.

Since we're treating this as a required field, we need to update the validation message from `zod` (by default it is just "Required"), Since we aren't dealing with validations on a `zod` type (like `.nonempty()`), there are a few possible errors, so we need to be explicit by defining a `required_error`:

```tsx
const formSchema = z.object({
  phone: looseOptional(mobilePhoneNumberSchema),
  email: z
    .string()
    .nonempty("Please specify an email")
    .email("Please specify a valid email"),
  preferredContactMethod: z.enum(contactMethods, {
    required_error: "Please select a contact method",
  }),
});
```

Nice, but you might be wondering... why did we specify `null` as a possible type of the `item` that's being passed to `onChange`? Try selecting a value, then hit the `X` on the right of the input to clear the selection, then loose focus of the field (remember we specified the revalidate mode of `onBlur` in the [previous tutorial](https://dev.to/timwjames/building-forms-with-zod-and-react-hook-form-2geg)). You'll see:

> Expected 'Email' | 'Phone', received null

which is not exactly user-friendly. That's because MUI sets the input to `null` when clearing the input, so from `zod`'s perspective, the value is still defined. Thankfully, `zod` catches this as an `invalid_type_error`, so we just need to update our schema (unfortunately with a bit of duplication):

```tsx
preferredContactMethod: z.enum(contactMethods, {
  required_error: 'Please select a contact method',
  invalid_type_error: 'Please select a contact method',
}),
```

You'll also notice an error being logged to the console. That's because MUI interprets a `value` being `undefined` as indicating that the component is uncontrolled. But then when the `value` changes to one of the `options`, MUI then needs to treat it as controlled, and this can be problematic ([here's why](https://legacy.reactjs.org/docs/forms.html#controlled-components)). To get around this, we can specify the default value of the component within our `useForm` hook as `null`, not `undefined`:

```tsx
defaultValues: {
  email: '',
  phone: '',
  preferredContactMethod: null,
},
```

An aside: the fact that the default/cleared value is `null` means that if you wanted to make this field optional, `.optional()` on the schema won't cover you (since that just checks if it is `undefined`). To solve this, you can `preprocess` the value to convert `null` to `undefined`. The helper function `looseOptional` we built in the [previous tutorial](https://stackblitz.com/edit/stackblitz-starters-1f3swh?file=src%2FApp.tsx) does precisely that.

## Updating the Schema with Dependent Fields

Nice, we have our `Autocomplete` field working, now we want to validate that if the user selects their preferred contact method as "Email", that they actually specify an email (ditto for "Phone").

Something you'll notice is that the validation of each field within the schema is independent of other fields in the schema. We need something else to make that validation happen... `superRefine` to the rescue!

[`superRefine`](https://zod.dev/?id=superrefine) allows us to apply validation on the overall `z.object()`, and gives us the current `value` of the object, and `context` that we can use to imperatively "inject" errors into specific fields which `react-hook-form` can then interpret.

First, let's make both `email` and `phone` optional by default, since they'll be required conditionally on `preferredContactMethod`:

```tsx
const formSchema = z.object({
  phone: looseOptional(mobilePhoneNumberSchema),
  email: looseOptional(
    z
      .string()
      .nonempty("Please specify an email")
      .email("Please specify a valid email")
  ),
  preferredContactMethod: z.enum(contactMethods, {
    required_error: "Please select a contact method",
    invalid_type_error: "Please select a contact method",
  }),
});
```

Then we can add the `superRefine` to the mix:

```tsx
const formSchema = z.object({
  ...
  }),
})
.superRefine((values, context) => {
  if (
    values.preferredContactMethod === "Email" && !values.email
  ) {
    context.addIssue({
      code: z.ZodIssueCode.custom,
      message: "Please specify an email",
      path: ["email"],
    });
  } else if (
    values.preferredContactMethod === "Phone" && !values.phone
  ) {
    context.addIssue({
      code: z.ZodIssueCode.custom,
      message: "Please specify a phone number",
      path: ["phone"],
    });
  }
});
```

Try selecting an option, then hit submit. 🤯 `context.addIssue` allows us to add an error to whatever `path` we want (be aware that `path` is only stringly typed), using `z.ZodIssueCode.custom` since we aren't using any of `zod`'s built-in errors (aka issues).

## Triggering Validation Conditionally

Cool, but you may have noticed that the message only updates when hitting the submit button. That's because `react-hook-from` doesn't know to re-render the `email` or `phone` fields when `preferredContactMethod` changes. We can leverage two things from `react-hook-form`:

- [`watch`](https://react-hook-form.com/docs/useform/watch) the `preferredContactMethod` for changes.
- [`trigger`](https://react-hook-form.com/docs/useform/trigger) validation of `email` and `phone`.

We can capture this logic as follows:

```tsx
const { control, handleSubmit, watch, trigger } = useForm<FormFields>({
  ...
});

useEffect(() => {
  const subscription = watch((_value, { name }) => {
    if (name === 'preferredContactMethod') {
      void trigger(['phone', 'email']);
    }
  });

  // Cleanup the subscription on unmount.
  return () => subscription.unsubscribe();
}, [watch, trigger]);
```

Now the user get immediate feedback based on their selection.

## Marking Fields as Required Conditionally

We also want to set the `email` or `phone` fields as `required` conditional on the selected `preferredContactMethod`. Sure, this is captured by the schema, but this isn't visible to the underlying MUI components.

We could use `watch` to achieve this as well, but we can optimise our re-renders using [`getValues`](https://react-hook-form.com/docs/useform/getvalues):

```tsx
const { control, handleSubmit, watch, trigger, getValues } =
  useForm<FormFields>({

...

<TextField
  name="phone"
  ...
  required={getValues('preferredContactMethod') === 'Phone'}
/>
```

and the equivalent for `email`.

And that's it, we now have a polished UX to conditionally validate different fields. 🚀

You can find the complete source code and an interactive demo on [Stackblitz](https://stackblitz.com/edit/stackblitz-starters-ylraa8?file=src%2FApp.tsx).

## Gotchas

If you're using `superRefine` in a larger form, where there are other unrelated fields to those you want to conditionally validate, you might notice some interesting behaviour. If those unrelated fields are invalid, the logic you defined in `superRefine` might not trigger, at least until those unrelated fields are made valid. That's because `zod` doesn't run `superRefine` until the validation within the `z.object()` passes, which is intentional, but can lead to an unexpected UX. A workaround is to defined those unrelated fields in a separate `z.object()`, then do an [`intersection`](https://zod.dev/?id=intersections) with the other `z.object()` with the `superRefine` to create the overall schema. Props to [this GitHub issue](https://github.com/colinhacks/zod/issues/479#issuecomment-853404300) for identifying that solution.

## Further Reading

- Previous post on building forms with `zod` and `react-hook-form`: <https://dev.to/timwjames/building-forms-with-zod-and-react-hook-form-2geg>
- [Creating Custom io-ts Decoders for Runtime Parsing](https://medium.com/agiledigital/creating-custom-io-ts-decoders-for-runtime-parsing-ffbfc8705bfa)
- [Parse, don’t validate](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/)
- Stay tuned for a follow up post on implementing nested fields.
