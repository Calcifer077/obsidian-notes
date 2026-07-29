---
title: Writing Markup with JSX
source: https://react.dev/learn/writing-markup-with-jsx
created: 2026-07-29
tags:
  - react
---
JSX is a syntax extension for JS that lets us write HTML-like markup inside a JS file. 

## JSX: Putting markup into JS

The Web has been built on HTML, CSS, and JavaScript. For many years, web developers kept content in HTML, design in CSS, and logic in JavaScript—often in separate files! Content was marked up inside HTML while the page’s logic lived separately in JavaScript:

![](../../../assets/Pasted%20image%2020260729165352.png)

But as the Web became more interactive, logic increasingly determined content. JavaScript was in charge of the HTML! This is why **in React, rendering logic and markup live together in the same place —components.**

![](../../../assets/Pasted%20image%2020260729165414.png)

Each react component is a JS function that may contain some markup that react renders into the browser. React components use a syntax extension called JSX to represent that markup.

## Converting HTML to JSX

Let's say we want to convert following HTML to JSX.

```HTML
<h1>Hedy Lamarr's Todos</h1>
<img
  src="https://react.dev/images/docs/scientists/yXOvdOSs.jpg"
  alt="Hedy Lamarr"
  class="photo"
>
<ul>
    <li>Invent new traffic lights
    <li>Rehearse a movie scene
    <li>Improve the spectrum technology
</ul>
```

There are some rules of converting HTML to JSX:

### The rules of JSX

#### 1. Return a single root element

To return multiple elements from a component, **wrap them with a single parent tag.**

For example, you can use a `<div>`:

```JSX
<div>
  <h1>Hedy Lamarr's Todos</h1>
  <img
    src="https://react.dev/images/docs/scientists/yXOvdOSs.jpg"
    alt="Hedy Lamarr"
    class="photo"
  >
  <ul>
    ...
  </ul>
</div>
```

If you don't want to add an extra `<div>` to your markup, you can write `<>` and `</>` instead:

```JSX
<>
  <h1>Hedy Lamarr's Todos</h1>
  <img
    src="https://react.dev/images/docs/scientists/yXOvdOSs.jpg"
    alt="Hedy Lamarr"
    class="photo"
  >
  <ul>
    ...
  </ul>
</>
```

This empty tag is called a _[Fragment.](https://react.dev/reference/react/Fragment)_ Fragments let you group things without leaving any trace in the browser HTML tree.

##### Why do multiple JSX tags need to be wrapped?

JSX looks like HTML, but under the hood it is transformed into plain JavaScript objects. You can’t return two objects from a function without wrapping them into an array. This explains why you also can’t return two JSX tags without wrapping them into another tag or a Fragment.

#### 2. Close all the tags

JSX requires tags to be explicitly closed: self-closing tags like `<img>` must become `<img />`, and wrapping tags like `<li>oranges` must be written as `<li>oranges</li>`.

#### 3. camelCase most of the things

JSX turns into JavaScript and attributes written in JSX become keys of JavaScript objects. In your own components, you will often want to read those attributes into variables. But JavaScript has limitations on variable names. For example, their names can’t contain dashes or be reserved words like `class`.

This is why, in React, many HTML and SVG attributes are written in camelCase. 

_Note: For historical reasons, `aria-*` and `data-*` attributes are written as in HTML with dashes._

## Recap

Now you know why JSX exists and how to use it in components:

- React components group rendering logic together with markup because they are related.
- JSX is similar to HTML, with a few differences. You can use a [converter](https://transform.tools/html-to-jsx) if you need to.
- Error messages will often point you in the right direction to fixing your markup.