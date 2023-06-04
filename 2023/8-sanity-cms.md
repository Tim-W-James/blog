Using a headless CMS provides developers with the flexibility they need, and exposes a user-friendly interface to clients that allows them to manage their own content. In this post, I'll go through the configuration of an SEO optimised app backed by a site builder with a realtime editor preview, achieved with NextJS and Sanity CMS.

## Overview

We'll be extending the [NextJS + SanityCMS blog starter](https://github.com/sanity-io/nextjs-blog-cms-sanity-v3), which gives us the following:

- Static site generation via Next js for a performant and SEO friendly site.
- Content is defined using a headless CMS (Sanity.io).
- Users can interact with the CMS via Sanity Studio deployed to `https://SITE_URL/studio`:
  - Draft mode with a side-by-side preview in Sanity Studio or live at `https://SITE_URL/api/preview`.
  - Secured by Sanity's build in auth.
  - Real-time collaboration.
- Incremental Static Revalidation via a webhook-trigger to publish new site content.

Instead of a blog, we'll be updating the schema to allow users to create their own general static site with:

- Any number of pages for their site:
  - Define content using a rich text editor:
    - Basic markdown-style formatting (headers, lists, links, etc.)
    - Embed images, files and PDFs which are optimisied and served by NextJS.
  - SEO title and description
- Custom nav bar with configurable routes for each page:
  - Uses NextJS's routing to provide optimised code splitting and client-side routing after the initial load.
  - Support for nested routes.

We'll be using Sanity Studio, which will provide our users with an interface to interact with the CMS and customise their site. Unlike other SaaS site builders, Sanity themselves don't host or manage Sanity Studio, it is essentially just a React app that uses their API. This means we can configure the studio however we want, and we are far less constrained than a service like SquareSpace or Wix. We'll be deploying the studio along side the app itself, at the `/studio` path.

## Defining the Entity Structure

We first need to customise how we handle each type of top-level entity in Sanity Studio. We have 3 kinds:

- "Documents" - multiple instances of an entity with their own configuration (**pages**):
  - Display these in a list, where each page automatically opens a preview showing that specific page.
  - Supporting the ability to create/delete each document.
- "Singletons" - global config where only 1 can exist (settings for the site title, nav, etc.):
  - Need to make sure only 1 of these entities exist, so disable creation/deletion.
- "Singleton with preview" - it makes sense to display a preview for certain singletons (the home page, since we need to ensure that exists at the root path).

This is achieved in [`sanity.config.ts`](https://github.com/Tim-W-James/cms-site-builder/blob/main/sanity.config.ts). The [`previewStructurePlugin`](https://github.com/Tim-W-James/nextjs-sanity/blob/aefbabe89d5f5dbcdc2cc51a655dc317223ce6b0/plugins/previewStructure.tsx#L11) function places the 2 variants of singletons Sanity Studio sidebar and configures the preview as needed. We also need to remove them from the global "new document" button, so we pass them to [`singletonPlugin`](https://github.com/Tim-W-James/nextjs-sanity/blob/aefbabe89d5f5dbcdc2cc51a655dc317223ce6b0/plugins/singeton.ts#L2-L3), then add the "open preview" button to the desired preview singletons via [`productionUrl`](https://github.com/Tim-W-James/nextjs-sanity/blob/aefbabe89d5f5dbcdc2cc51a655dc317223ce6b0/plugins/productionUrl/index.ts#L11). All the other documents (just pages here) go to [`previewDocumentNode`](https://github.com/Tim-W-James/nextjs-sanity/blob/aefbabe89d5f5dbcdc2cc51a655dc317223ce6b0/plugins/previewPane/index.tsx#L5).

## Defining the Schemas

You can browse the source code for the schemas I'll be discussing [here](https://github.com/Tim-W-James/nextjs-sanity/blob/5e2267521e80c1b262f1aea80f02002079460b86/schemas), and check [Sanity's documentation](https://www.sanity.io/docs/schema-types) if you want to add your own.

### Pages

First, define a page schema:

```ts
export default defineType({
  name: "pages",
  title: "Pages",
  icon: DocumentsIcon,
  type: "document",
  preview: {
    select: { title: "metadata.title", subtitle: "metadata.description" },
  },
  fields: [
    defineField({
      name: "metadata",
      title: "Metadata",
      type: "pageMeta",
      validation: (rule) => rule.required(),
    }),
    defineField({
      name: "content",
      title: "Content",
      type: "page",
    }),
  ],
});
```

[View full source](https://github.com/Tim-W-James/nextjs-sanity/blob/aefbabe89d5f5dbcdc2cc51a655dc317223ce6b0/schemas/pages/pages.ts#L4)

A [`document`](https://www.sanity.io/docs/document-type) type is the fundamental building block of Sanity, which allows us to configure arbitrary fields. I've split this into two field 'groups' - metadata (title, URL path, etc.), and the page content itself. We also infer the preview information (which appears in Sanity Studio) from the metadata.

The metadata schema is as follows:

```ts
export default defineType({
  name: "pageMeta",
  title: "Page Metadata",
  icon: DocumentsIcon,
  type: "object",
  preview: { select: { title: "title", subtitle: "description" } },
  fields: [
    defineField({
      name: "title",
      title: "Page Title",
      description: "Appears in the browser tab and search results.",
      type: "string",
      validation: (rule) => rule.required(),
    }),
    defineField({
      name: "description",
      description: "Appears in search results.",
      title: "Description",
      type: "string",
      validation: (rule) => rule.max(155),
    }),
    defineField({
      name: "path",
      title: "Path",
      description: `Used for the URL path: https://example.com/your-path`,
      type: "slug",
      options: {
        source: "metadata.title",
        isUnique: (value, context) => context.defaultIsUnique(value, context),
      },
      validation: (rule) => rule.required(),
    }),
  ],
});
```

[View full source](https://github.com/Tim-W-James/nextjs-sanity/blob/5e2267521e80c1b262f1aea80f02002079460b86/schemas/pageMeta.ts#L4)

We have a few standard fields here, which you can read more about [here](https://www.sanity.io/docs/schema-types). The `path` is the more interesting one:

- Type `slug` ensures that the path is unique (since it'll be part of the URL) across other pages, and has a valid format.
- We infer the default value from the title, which is automatically converted into a valid format for the `slug`.

Now to define the content:

```ts
export default defineType({
  name: "page",
  title: "Page",
  icon: DocumentsIcon,
  type: "object",
  preview: { select: { title: "header" } },
  fields: [
    defineField({
      name: "header",
      title: "Header",
      description: "Appears at the top of the page.",
      type: "string",
    }),
    defineField({
      name: "body",
      title: "Body",
      description:
        "Rich text content, including sub headings, links, images, and PDFs.",
      ...
    }),
  ],
});
```

Now for the interesting part - we need a way for the user to customise the content of the page in a familiar editor-like experience

### Rich Text Blocks

Raw markdown is an option, but instead we'll be using [portable text](https://www.sanity.io/docs/presenting-block-text) to support some custom object types.

Out of the box, Sanity gives us a [`block`](https://www.sanity.io/docs/block-type) type for rich text. That gives us basic formatting options (decorators like bold, italics, etc.), but we want to extend this with a few things. You can follow along with the source code [here](https://github.com/Tim-W-James/nextjs-sanity/blob/aefbabe89d5f5dbcdc2cc51a655dc317223ce6b0/schemas/page.ts#L23). It must be used with an array so we can define custom objects to appear within the page:

```ts
defineField({
  name: "body",
  title: "Body",
  description:
    "Rich text content, including sub headings, links, images, and PDFs.",
  type: "array",
  of: [
    {
      type: "block",
      marks: {
        ...
      },
    },
    ...
  ],
}),
```

We'll tackle links first, allowing the user to highlight text, and convert it into a link. We can use `annotations`, which allow us to create an arbitrary object associated with some text. An annotation for a link to an external URL looks like this:

```ts
{
  name: "urlLink",
  type: "object",
  title: "URL Link",
  description: "Link to an external URL.",
  icon: LinkIcon,
  fields: [
    {
      name: "url",
      type: "url",
      title: "URL",
      validation: (rule) => rule.uri().required(),
    },
    {
      name: "hoverText",
      type: "string",
      title: "Description Text",
    },
    {
      name: "shouldUseNewTab",
      type: "boolean",
      title: "Open link in new tab.",
      initialValue: true,
    },
  ],
}
```

Later, we'll decide how to use these attributes to render this link, but you can imagine how these could translate into an HTML anchor element.

A few things to note:

- We provide some helpful descriptions and icons to determine how Sanity Studio displays this object to our user (who doesn't need to be particularly tech savvy).
- We also provide some validation for the format of the URL. You can read more about the validation options Sanity provides [here](https://www.sanity.io/docs/validation).

Similarly, we can create a link for a file:

```ts
{
  name: "fileLink",
  type: "object",
  title: "File Link",
  description: "Link pieces of text to a file.",
  icon: DocumentTextIcon,
  fields: [
    {
      name: "file",
      type: "file",
      title: "File Attachment",
      validation: (rule) => rule.required(),
    },
    {
      name: "fileName",
      type: "string",
      title: "File Name",
    },
  ],
},
```

The [`file`](https://www.sanity.io/docs/file-type) type does a lot of the heavy lifting here, allowing the user to upload a file of their choosing, which Sanity will then serve to the user.

We also want to allow users to embed images in their site. We don't use an annotation for this, since images are independent from text, so it is a separate element in the `array` of the parent `body` field:

```ts
{
  type: "object",
  name: "embeddedImage",
  title: "Image",
  icon: ImageIcon,
  fields: [
    {
      name: "imageFile",
      type: "image",
      title: "Image File",
      validation: (rule) => rule.required(),
    },
    {
      name: "imageDescription",
      type: "string",
      title: "Description",
      description: "If the image fails to load, this text will appear.",
      validation: (rule) => rule.required(),
    },
    {
      name: "imageWidth",
      type: "number",
      title: "Image Size",
      description:
        "Pixel width of the image, leave empty for auto scaling.",
      validation: (rule) => rule.max(5000).min(10),
    },
    {
      name: "wrapText",
      type: "boolean",
      title: "Wrap Text",
      description: "Wrap text around the image.",
      initialValue: false,
    },
  ],
},
```

Sanity has a built-in [`image`](https://www.sanity.io/docs/image-type) type too, but we need the user to specify additional fields so we can determine how to render the image in the frontend later.

Let's create an embedded file too:

```ts
{
  type: "object",
  name: "embeddedFile",
  title: "File",
  icon: DocumentPdfIcon,
  fields: [
    {
      name: "file",
      type: "file",
      title: "File Attachment",
      validation: (rule) => rule.required(),
    },
    {
      name: "fileName",
      type: "string",
      title: "File Name",
    },
    {
      name: "shouldRenderPdf",
      type: "boolean",
      title: "Render PDF",
      description: "If the file is a PDF, display it on the page.",
      initialValue: true,
    },
    {
      name: "pdfHeight",
      type: "number",
      title: "PDF Display Height",
      description:
        "Height for the PDF display, leave empty for auto scaling.",
      hidden: ({ parent }) => !parent?.shouldRenderPdf,
      validation: (rule) => rule.max(5000).min(50),
    },
  ],
},
```

We also give the user an option to embed a PDF file, which we'll use to render the PDF within an `iframe`. Since we have a field that is specific to whether the PDF is rendered or not, we can use the `hidden` option.

All together, that gives us a rich text editor with some custom objects.

[View full source](https://github.com/Tim-W-James/nextjs-sanity/blob/aefbabe89d5f5dbcdc2cc51a655dc317223ce6b0/schemas/page.ts#L10)

### Defining the Nav

```ts
export default defineType({
  name: "navigation",
  title: "Navigation",
  icon: MasterDetailIcon,
  type: "document",
  preview: {
    prepare: () => ({ title: "Navigation" }),
  },
  fields: [
    {
      name: "items",
      title: "Nav Items",
      type: "array",
      of: [
        {
          type: "object",
          name: "menuItem",
          fields: [
            {
              name: "title",
              title: "Title",
              type: "string",
              validation: (rule) => rule.required(),
            },
            {
              name: "routes",
              title: "Routes",
              description:
                "Pages for the nav item. Define multiple routes for a" +
                " dropdown menu.",
              type: "array",
              of: [
                {
                  type: "object",
                  name: "subMenuItem",
                  fields: [
                    {
                      name: "title",
                      title: "Title",
                      description:
                        "If this is your only route, leave blank." +
                        " Otherwise used for the dropdown menu.",
                      type: "string",
                    },
                    {
                      name: "page",
                      title: "Page",
                      type: "reference",
                      to: [{ type: "pages" }],
                      validation: (rule) => rule.required(),
                    },
                  ],
                },
              ],
            },
          ],
        },
      ],
    },
  ],
});
```

[View full source](https://github.com/Tim-W-James/nextjs-sanity/blob/aefbabe89d5f5dbcdc2cc51a655dc317223ce6b0/schemas/nav.ts#L4)

We want our navbar to support a dropdown for nested routes, so we have 2 levels of arrays. Using [`reference`](https://www.sanity.io/docs/reference-type), the user can easily search for the pages they created.

For the sake of brevity, I'll leave out the other schemas, you can browse them [here](https://github.com/Tim-W-James/nextjs-sanity/blob/5e2267521e80c1b262f1aea80f02002079460b86/schemas).

## Building Out the React Frontend

With Sanity being a headless CMS, we have the freedom to render our components and style the site however we want. I'll speed through some of the logic here to get to the interesting parts (you can find the general approach in the [NextJS + SanityCMS blog starter](https://github.com/sanity-io/nextjs-blog-cms-sanity-v3)).

Since we're using NextJS and want static site generation, data from the CMS comes from `getStaticProps`. Thankfully, this means we don't need to deal with any loading states, since the site will be rendered before it reaches the client. Nice.

A reusable function [`getStaticPageProps`](https://github.com/Tim-W-James/nextjs-sanity/blob/46000ffbb52943379591c93f364ce849879a4009/lib/getStaticPageProps.ts#L25) allows us to query the page data for any `path`. This runs the following query:

```ts
export const pageQuery = (path: string) =>
  groq`*[_type == "pages" && metadata.path.current == "${path}"][0]`;
```

[`groq`](https://www.sanity.io/docs/groq) is Sanity's custom query language. We won't go into detail for it here, but you can treat it a bit like GraphQL. One useful thing to note: Sanity Studio has a built-in playground called ["Vision"](https://www.sanity.io/docs/the-vision-plugin) which is handy for development.

The page data ultimately ends up as props in the [PageLayout](https://github.com/Tim-W-James/nextjs-sanity/blob/aefbabe89d5f5dbcdc2cc51a655dc317223ce6b0/components/PageLayout.tsx#L19) component. We also need to know if the page is being viewed as a `preview` since it that case the site isn't pre-rendered and we need to handle a loading state.

```tsx
const PageLayout = (props: PageLayoutProps) => {
  const {
    preview,
    loading,
    page: { content, metadata },
    settings,
    routes,
  } = props;

  return (
    <>
      <PageHead pageMeta={metadata} settings={settings} />

      <Layout loading={loading} preview={Boolean(preview)}>
        <Navigation routes={routes} />
        <Container>
          <h1 className={clsx("pt-3")}>{content?.header ?? "Heading"}</h1>
          <hr />
          {content?.body ? (
            <PortableText
              components={PortableTextRenderer}
              value={content.body}
            />
          ) : (
            "Body"
          )}
        </Container>
      </Layout>
    </>
  );
};
```

We also treat these props as optional, since even though they are required, the user configuring the CMS data in Sanity Studio can view the realtime preview before that data is valid, and it's preferable that they see the empty 'structure' of the site rather than an error. We also don't have type safety for the schema we defined for Sanity, so I've had to define my own types for these props... (more on this later).

The complex part here is `PortableText`, so let's give into that next.

### Portable Text

Remember all those objects we defined for the rich text editor? Now we can decide how to actually render them.

The magic happens in the [`PortableTextRenderer`](https://github.com/Tim-W-James/nextjs-sanity/blob/aefbabe89d5f5dbcdc2cc51a655dc317223ce6b0/components/portableText/PortableTextRenderer.tsx#L9):

```ts
const PortableTextRenderer: PortableTextComponents = {
  marks: {
    em: ({ children }) => <em>{children}</em>,
    strong: ({ children }) => <strong>{children}</strong>,
    u: ({ children }) => <u>{children}</u>,
    code: ({ children }) => <code>{children}</code>,
    urlLink: UrlLink,
    fileLink: FileLink,
  },
  list: {
    bullet: ({ children }) => <ul>{children}</ul>,
    number: ({ children }) => <ol>{children}</ol>,
  },
  listItem: {
    bullet: ({ children }) => <li>{children}</li>,
    number: ({ children }) => <li>{children}</li>,
  },
  block: {
    blockquote: ({ children }) => (
      <blockquote className={clsx("blockquote")}>{children}</blockquote>
    ),
  },
  types: {
    embeddedImage: ImageComponent,
    embeddedFile: FileComponent,
  },
};
```

Breaking that down piece by piece:

- We are essentially defining React components to render
- `marks` (the `annotations` and `decorations` for text) wrap `children` (the text) in some basic HTML like `em` for bold, etc. This also includes our custom URL and file links.
- `list`, `listItem` and `block` map to their HTML counterparts.
- Our custom `types` for images and files.

### URL Links

The `UrlLink` looks like this:

```tsx
const { url, hoverText, shouldUseNewTab } = parsedUrlResult.data;
return (
  <a
    href={url}
    rel="noreferrer"
    target={shouldUseNewTab ? "_blank" : "_self"}
    title={hoverText}
  >
    {children}
  </a>
);
```

We map the user defined options from the Sanity schema into a link. Before we do that however, we need to validate the input from the API is what we expect. Again, we don't know if the data specified in preview mode is valid (initially it won't be), and we don't want the preview to crash, instead it should give some visual feedback to the user.

We'll be using [`zod`](https://github.com/colinhacks/zod) to do that validation at runtime:

```tsx
const urlLinkSchema = z.object({
  _key: z.string(),
  _type: z.literal("urlLink"),
  url: z.string(),
  hoverText: z.string().optional(),
  shouldUseNewTab: z.boolean().optional(),
});

...

const parsedUrlResult = urlLinkSchema.safeParse(value);
if (!parsedUrlResult.success) {
  console.error("Url object did not match schema", {
    cause: parsedUrlResult.error,
  });
  return <span className={clsx("text-danger")}>{children}</span>;
}
```

[View full source](https://github.com/Tim-W-James/nextjs-sanity/blob/aefbabe89d5f5dbcdc2cc51a655dc317223ce6b0/components/portableText/UrlLink.tsx#L13)

Note that we need to be careful our Sanity native validation and frontend validation align here - the user in Sanity Studio shouldn't be able to save their changes and exit draft mode while the frontend is unable to render the content correctly. Unfortunately, Sanity doesn't provide any tooling to generate `zod` schemas from their native validation, or even type definitions more generally for our frontend to consume, so we have to write them manually. As an aside, this might be my biggest gripe with Sanity overall, type safety is not a priority of theirs unfortunately.

### File Links

`FileLink` is similar, but we need to construct a URL to the file that has been uploaded on Sanity:

```tsx
const fileData = parsedFileResult.data;
const { fileName } = fileData;
const [_file, id, extension] = fileData.file.asset._ref.split("-");
return (
  <a
    href={`${sanityBaseUrl}/${id}.${extension}?dl=${
      fileName ? encodeURIComponent(fileName) : id
    }.${extension}`}
  >
    {children}
  </a>
);
```

[View full source](https://github.com/Tim-W-James/nextjs-sanity/blob/aefbabe89d5f5dbcdc2cc51a655dc317223ce6b0/components/portableText/FileLink.tsx#L19)

### Embedded Files and PDFs

Since we give the user the option to embed PDF files, we do this conditionally with an `iframe`:

```tsx
const { fileName, shouldRenderPdf, pdfHeight, file } = parsedFileResult.data;
const [_file, id, extension] = file.asset._ref.split("-");
return extension === "pdf" && shouldRenderPdf ? (
  <iframe
    height={pdfHeight}
    src={`${sanityBaseUrl}/${id}.${extension}`}
    title={fileName || "PDF"}
    width="100%"
  />
) : (
  <a
    href={`${sanityBaseUrl}/${id}.${extension}?dl=${
      fileName ? encodeURIComponent(fileName) : id
    }.${extension}`}
  >
    {`${fileName || "file"}.${extension}` || "Download file"}
  </a>
);
```

[View full source](https://github.com/Tim-W-James/nextjs-sanity/blob/aefbabe89d5f5dbcdc2cc51a655dc317223ce6b0/components/portableText/FileComponent.tsx#L20)

### Images

Now for the tricky one: we want to serve the image in an optimised way using [NextJS's Image](https://nextjs.org/docs/pages/api-reference/components/image). This gives us:

- **Size Optimization** - Automatically serve correctly sized images for each device, using modern image formats like WebP and AVIF.
- **Visual Stability** - Prevent layout shift automatically when images are loading.
- **Faster Page Loads** - Images are only loaded when they enter the viewport using native browser lazy loading, with optional blur-up placeholders.
- **Asset Flexibility** - On-demand image resizing, even for images stored on remote servers.

while serving the image directly from Sanity does not.

Thankfully, someone has already built a library to do the heavy lifting for us - [`next-sanity-image`](https://www.npmjs.com/package/next-sanity-image) - which gives us a few helper functions:

```tsx
const ImageRenderer: React.FC<{
  image: ImageType;
  isInline: boolean;
  sanityClient: SanityClient;
}> = ({ image, isInline, sanityClient }) => {
  const imageProps = useNextSanityImage(sanityClient, image.imageFile);
  const { width, height } = getImageDimensions(image.imageFile);
  return (
    <Image
      {...imageProps}
      alt={image.imageDescription || ""}
      loading="lazy"
      style={{
        // Display alongside text if image appears inside a block text span
        display: isInline ? "inline-block" : "block",
        float: image.wrapText ? "left" : "none",

        // Avoid jumping around with aspect-ratio CSS property
        aspectRatio: width / height,
      }}
      width={image.imageWidth || width}
    />
  );
};
```

[View full source](https://github.com/Tim-W-James/nextjs-sanity/blob/aefbabe89d5f5dbcdc2cc51a655dc317223ce6b0/components/portableText/ImageComponent.tsx#L51)

And boom, we don't need to worry about an external service like Cloudinary; NextJS covers all our images optimisation needs.

## Deployment

There are a few other components you can explore, like the [`nav`](https://github.com/Tim-W-James/Journeys-Continue/blob/1912ecdd6aafc777862d143ae075b7b087ec804b/components/Nav.tsx#L10), but we now have a decently versatile page builder.

You can deploy this app to Vercel using the [template](https://github.com/Tim-W-James/nextjs-sanity), and create your own Sanity data lake (the free tier is pretty generous).

You'll want to configure incremental static regeneration to ensure published changes to CMS data are automatically pushed to the main site (see this [file](https://github.com/Tim-W-James/Journeys-Continue/blob/6cfc332dbe02d707036cf8047d0703ccf7d06d9f/pages/api/revalidate.ts) - be aware that if you update the schema you'll also need to update the logic that handles that revalidation). This allows you to trigger re-validations on only data which has changed, rather than rebuilding the entire app.

## Wrapping Up

Again, you can view the full source code [here](https://github.com/Tim-W-James/nextjs-sanity), which is built with minimal Bootstrap components. Since the CMS is headless, you can style the frontend however you need, or switch to another component library like Material UI - whatever you want really. This makes the template easily adaptable to different clients, you can restyle the app to a brand and grant your client access to Sanity Studio for a minimal maintenance burden. Take a look at this template in action on a full project [here](https://github.com/Tim-W-James/Journeys-Continue).
