---
title: Your first component
source: https://react.dev/learn/your-first-component
created: 2026-07-29
tags:
  - react
---
## Components: UI building blocks

On the web, HTML lets us create rich structured documents with its built-in set of tags like `<h1>` and `<li>`.

```HTML
<article>
  <h1>My First Component</h1>
  <ol>
    <li>Components: UI Building Blocks</li>
    <li>Defining a Component</li>
    <li>Using a Component</li>
  </ol>
</article>
```

Markup like this, combined with CSS for style, and JavaScript for interactivity, lies behind every sidebar, avatar, modal, dropdown—every piece of UI you see on the Web.

React lets you combine your markup, CSS, and JavaScript into custom “components”, **reusable UI elements for your app.** Under the hood, it still uses the same HTML tags like `<article>`, `<h1>`, etc.

Just like with HTML tags, we can compose, order and nest components to design whole pages.

## Defining a component

A react component is a JS function that you can _sprinkle with markup._

```jsx
export default function Profile() {
  return (
    <img     src="https://react.dev/images/docs/scientists/MK3eW3Am.jpg"
      alt="Katherine Johnson"
    />
  )
}
```

Here's how to build a component:

### Step 1: Export the component

The `export default` prefix lets us export main function from a file which we can later import in other files.

### Step 2: Define the function

With `function Profile() { }` you define a JavaScript function with the name `Profile`.

_Note: React components name should start with capital letter._

### Step 3: Add markup

The component above returns an `<img />` tag which is written like HTML, but it is actually JS under the hood. This syntax is called **JSX**, and it lets us embed markup inside JS.

We need to return this markup. It is more than one line we need to use parentheses. 

## Using a component

Now that we've defined our `Profile` component, we can nest it inside any other component and use it. 

```JSX
function Profile() {
  return (
    <img
      src="https://react.dev/images/docs/scientists/MK3eW3As.jpg"
      alt="Katherine Johnson"
    />
  );
}

export default function Gallery() {
  return (
    <section>
      <h1>Amazing scientists</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

## What the browser sees

Notice the difference in casing:

- `<section>` is lowercase, so React knows we refer to an HTML tag.
- `<Profile />` starts with a capital `P`, so React knows that we want to use our component called `Profile`.

And `Profile` contains even more HTML: `<img />`. In the end, this is what the browser sees:

```JSX
<section>
  <h1>Amazing scientists</h1>
  <img src="https://react.dev/images/docs/scientists/MK3eW3As.jpg" alt="Katherine Johnson" />
  <img src="https://react.dev/images/docs/scientists/MK3eW3As.jpg" alt="Katherine Johnson" />
  <img src="https://react.dev/images/docs/scientists/MK3eW3As.jpg" alt="Katherine Johnson" />
</section>
```

_Note: Components can render other components, but we must never nest their definitions._

## Recap

You’ve just gotten your first taste of React! Let’s recap some key points.

- React lets you create components, **reusable UI elements for your app.**
    
- In a React app, every piece of UI is a component.
    
- React components are regular JavaScript functions except:
    
    1. Their names always begin with a capital letter.
    2. They return JSX markup.