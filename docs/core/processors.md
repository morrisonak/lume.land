---
title: Processors
description: A guide on extending Lume with custom processors
order: 15
---

${toc}

A processor is a function to transform the content of pages. Let's see an
example of a processor to minify HTML pages:

```js
function minifyHTML(page) {
  page.content = minify(page.content);
}
```

If you want to use this processor to build your site, you can register it in the
`_config.js` file:

```js
site.process([".html"], minifyHTML);
```

Now, all HTML pages with be minified.

## The page data

As you can see in the previous example, the function receives an object with the
page (or asset). This means that you can access not only to the page content but
to many other data:

```js
function process(page) {
  page.content; // The content of the page
  page.document; // The parsed HTML code, to use the DOM API
  page.src; // The info about the source file of this page
  page.dest; // The info about the destination of the page
  page.path; // The path of the output file (without the extension)
  page.ext; // The extension of the output file
  page.data; // All data available for this page (frontmatter merged with _data)
}
```

For example, let's say you only want to minify the pages with the value `minify`
as `true`:

```js
site.process([".html"], (page) => {
  if (page.data.minify) {
    page.content = minify(page.content);
  }
});
```

## Using the DOM API

You can use the **DOM API** (provided by
[deno-dom](https://github.com/b-fuze/deno-dom)) with methods like
`querySelector`, `setAttribute`, etc to modify HTML code. For example, let's
create a processor to add automatically the `alt` attribute to all images:

```js
site.process([".html"], (page) => {
  page.document.querySelectorAll("img").forEach((img) => {
    if (!img.hasAttribute("alt")) {
      img.setAttribute("alt", "This is a random alt");
    }
  });
});
```

Processors are executed just after render the page (with a template engine).
{.tip}

## Process assets

For non-HTML pages (like CSS or JavaScript files), you can use the processors to
compile CSS, minify JavaScript code or minify images.

```js
site.process([".js"], function (page) {
  page.content = myBundler(page.content);

  // Append .min to the filename
  // so it will be saved as example.min.js
  page.path += ".min";
});
```

Make sure the file extension that you want to process is previously loaded. See
[Loaders](loaders.md) for more information about how to register a new loader.
{.tip}

## Preprocess

If you need to execute a function **before rendering** (for example, to
configure a custom template engine or add extra data to some pages), you can use
a **preprocessor**. Preprocessors work like processors but with they are
executed before rendering.

Let's create a preprocessor to include a variable with the source filename:

```js
site.preprocess(
  [".html"],
  (page) => page.data.filename = page.src.path + page.src.ext,
);
```

## How processors and preprocessors work

Both processors and preprocessors are tied to file extensions (`.html`, `.js`
etc). To decide if a page must use a registered processor or preprocessor, Lume
search this extension in the input file (like `.md` or `.njk`) or the output
file (like `.html` or `.css`).

Another interesting thing is they are executed in the same order they are
defined. This allows to chain different processors to the same file extension.
For example: two processors to the `.css` extension, one to compile the code and
other to minify.

## Global (pre)processors

If you want to run a processor or preprocessor with all pages, use `*` in the
first argument:

```js
site.process("*", processAllPages);
```
