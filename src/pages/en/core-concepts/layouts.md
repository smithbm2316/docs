---
layout: ~/layouts/MainLayout.astro
title: Layouts
description: An intro to layouts, a type of Astro component that is shared between pages for common layouts.
i18nReady: true
---

**Layouts** are a special type of [Astro component](/en/core-concepts/astro-components/) useful for creating reusable page templates.

A layout component is conventionally used to provide an [`.astro` or `.md` page](/en/core-concepts/astro-pages/) both a **page shell** (`<html>`, `<head>` and `<body>` tags) and a `<slot />` to specify where in the layout page content should be injected.

Layouts often provide common `<head>` elements and common UI elements for the page such as headers, navigation bars and footers.

Layout components are commonly placed in a `src/layouts` directory in your project.

## Sample Layout

**`src/layouts/MySiteLayout.astro`**

```astro
---
---
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>My Cool Astro Site</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
  </head>
  <body>
    <nav>
      <a href="#">Home</a>
      <a href="#">Posts</a>
      <a href="#">Contact</a>
    </nav>
    <article>
      <slot /> <!-- your content is injected here -->
    </article>
  </body>
</html>
```

**`src/pages/index.astro`**

```astro {2} /</?MySiteLayout>/
---
import MySiteLayout from '../layouts/MySiteLayout.astro';
---
<MySiteLayout>
  <p>My page content, wrapped in a layout!</p>
</MySiteLayout>
```


📚 Learn more about [slots](/en/core-concepts/astro-components/#slots).

## Markdown Layouts

Page layouts are especially useful for [Markdown files](/en/guides/markdown-content/#markdown-and-mdx-pages). Markdown files can use a special `layout` property at the top of the frontmatter to specify which `.astro` component to use as a page layout.

**`src/pages/posts/post-1.md`**

```markdown {2}
---
layout: ../../layouts/BlogPostLayout.astro
title: Astro in brief
author: Himanshu
description: Find out what makes Astro awesome!
--- 
This is a post written in Markdown.
```

When a Markdown file includes a layout, it passes a `frontmatter` property to the `.astro` component which includes the frontmatter properties and the final HTML output of the page.


**`src/layouts/BlogPostLayout.astro`**

```astro /frontmatter(?:.\w+)?/
---
const {frontmatter} = Astro.props;
---
<html>
  <!-- ... -->
  <h1>{frontmatter.title}</h1>
  <h2>Post author: {frontmatter.author}</h2>
  <p>{frontmatter.description}<p>
  <slot /> <!-- Markdown content is injected here -->
   <!-- ... -->
</html>
```

📚 Learn more about Astro’s Markdown support in our [Markdown guide](/en/guides/markdown-content/).

## Nesting Layouts

Layout components do not need to contain an entire page worth of HTML. You can break your layouts into smaller components, and then reuse those components to create even more flexible, powerful layouts in your project.

For example, a common layout for blog posts may display a title, date and author. A `BlogPostLayout.astro` layout component could add this UI to the page and also leverage a larger, site-wide layout to handle the rest of your page.

**`src/layouts/BlogPostLayout.astro`**

```astro {2} /</?BaseLayout>/
---
import BaseLayout from './BaseLayout.astro'
const {frontmatter} = Astro.props;
---
<BaseLayout>
  <h1>{frontmatter.title}</h1>
  <h2>Post author: {frontmatter.author}</h2>
  <slot />
</BaseLayout>
```

## Base Layouts

A lot of people new to Astro and within the Astro community commonly ask questions like the following:

- Where do I put my global CSS file?
- Where do I load my custom fonts?
- Where do I add SEO tags?

A lot of these questions can be answered by addressing some of the ideas that the community has had surrounding creating a "Base Layout". When you're creating new pages, no matter how different your Astro pages are, there will be *some* overlap in the HTML tags that you'll be adding to your `<head>` and likely the `<body>` too.

### One base layout to rule them all...

One potential way of tackling how to set up your layouts is to create one main Base Layout that wraps all of your other layouts. Let's create a new file in our `src/layouts` directory called `BaseLayout.astro`.

Let's imagine that we have a few CSS stylesheets that we want to import on every page. For example, we might have a file called `reset.css` that has a CSS Reset, a stylesheet called `my-custom-font.css` that loads a custom local font, and a file called `type-system.css` that has some initial styles setting up say a type system that we generated using a tool like [Type Scale](https://type-scale.com/). We'll say that all of these files are located in our `src/styles` folder. In order to load them on every page, we'll include them in our `BaseLayout.astro` component which will wrap every a page on our site, so that we only have to declare al of this one time.

**`src/layouts/BaseLayout.astro`**

```astro
---
// located in src/styles
import '../styles/reset.css';
import '../styles/my-custom-font.css';
import '../styles/type-system.css';
---
```

We will import all of our stylesheets in the order in which we would like the browser to load them here in the code fence of our template so that we can let Astro process our CSS files in order to do minification at build time to send the most performant versions of these files to our users (this uses Vite's CSS processing under the hood)!

After we've handled importing our CSS files, we should set up some inital HTML that every page should use, since we don't want to have to add the `<html></html>` tag to every page that we create. We can pre-populate our `<head>` tag with some common `<meta>` tags that every page will need, and we can add our first prop to this `BaseLayout.astro` component, where we will accept a `title` string so that we can dynamically define the title of each page in each of our `.astro` files in our `pages` folder. Finally, we'll add our `<slot />` component in the `<body>` of our component as well, so that we keep this file as minimal and unopinionated as possible. All we want to do is load the global CSS resources, custom fonts, etc and set up the minimal amount of HTML that the browser needs. 

**`src/layouts/BaseLayout.astro`**

```astro
---
import '../styles/reset.css';
import '../styles/my-custom-font.css';
import '../styles/type-system.css';

const { title } = Astro.props;
---
<!-- src/layouts/BaseLayout.astro -->

<!DOCTYPE html>
<html lang='en'>
  <head>
    <meta charset='UTF-8' />
    <meta
      name='viewport'
      content='width=device-width,initial-scale=1'
    />
    <link
      rel='icon'
      type='image/svg+xml'
      href='/favicon.svg'
    />
    <meta
      name='generator'
      content={Astro.generator}
    />
    <title>{title}</title>
  </head>
  <body>
    <slot />
  </body>
</html>
```

From there, we can start using this `BaseLayout` to compose all of our other layouts to make it simpler to define layouts that will be more specific to the different pages on our site, with less boilerplate needed!
