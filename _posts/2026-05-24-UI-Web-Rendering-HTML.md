---
layout: post
title: "UI: Web Rendering: HTML → Render Tree"
date: 2026-05-24
math: false
---

# UI: Web Rendering: HTML → Render Tree

## HTML

HTML (HyperText Markup Language) "defines the meaning and structure of web content."[^1] So rather than just including the text content, or including also the styling, it includes the content and the *meaning* of what the element should be understood as. For example, it will designate something as a button, rather than a box, or as a heading rather than a large text.

The hypertext means some of the text can be links to other web pages or else, which is the main interactivity of the web. 

HTML "elements" are created using "tags" which are opened by `<` and closed by `>`. HTML also includes texts and comments and an HTML document is rendered into a DOM tree structure, which has nodes of those three types: element, text, comment. Element tags (and nodes) can also have attributes which are included properties. 

Here is the full list of [tags][mdn-tags] and [attributes][mdn-attributes].

## HTML DOM

The HTML DOM tree is a tree structure with element nodes (coming from HTML elements), text nodes, comment nodes, and the document node which is the root. "The DOM" is an [API][mdn-html-dom] that represents and interacts with the DOM tree representation.

Here is a practical example:

```html
<!doctype html>
<html>
  <head>
    <link rel="stylesheet" href="styles.css" />
  </head>
  <body>
    <h1 class="title">Hello</h1>
    <p id="intro">Welcome!</p>
  </body>
</html>
```

parses into the approximate DOM:
```
document
└── html
    ├── head
    │   └── link
    └── body
        ├── h1
        │   └── text
        └── p
            └── text
```

Like with programming language compilers, the HTML code gets parsed into an AST with a tokenizer and tree constructor.

## CSSOM

The CSS object model is another tree, independent of the HTML DOM, built from the imported CSS rules. It is relatively flat as a tree, mostly listing styles for selectors and occasionally increasing level with `@` rules like @media, @supports, or @layer. So for example:

```css
body {
  font-family: sans-serif;
}

@media (max-width: 600px) {
  h1 {
    font-size: 24px;
  }

  .title {
    color: red;
  }
}
```

approximately parses into:
```txt
CSSStyleSheet
├── CSSStyleRule: body
│   └── declarations
│       └── font-family: sans-serif
└── CSSMediaRule: @media (max-width: 600px)
    ├── CSSStyleRule: h1
    │   └── declarations
    │       └── font-size: 24px
    └── CSSStyleRule: .title
        └── declarations
            └── color: red
```

## Render Tree

The HTML DOM and CSSOM are combined into a computed render tree. Construction starts with the root and is traversed along paths to the leafs, ignoring non-visible nodes and picking up styles with most recent overwriting previous when there are conflicts. The browser then goes through further graphical steps to finally paint the pixels.

Notable libraries for manipulating the webpage into interactive apps include jQuery, which directly changes the HTML DOM with selectors and queries, and React, which tries to be more platform-agnostic by having its own "React element trees" which then a specific library `react-dom` converts to updates in the HTML DOM. Other renders can be hooked up instead of browsers, such as in React Native or a terminal renderer like in Ink.

[^1]: [MDN HTML](https://github.com/mdn/content/blob/82eeb8918aee4fe017a6ef1cb286bfd0a8a0ff43/files/en-us/web/html/index.md)

[mdn-tags]: https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements

[mdn-attributes]: https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes

[mdn-html-dom]: https://developer.mozilla.org/en-US/docs/Web/API/HTML_DOM_API