---
title: Importing and Exporting components
source: https://react.dev/learn/importing-and-exporting-components
created: 2026-07-29
tags:
  - react
---
## The root component file

In [Your first component](Your%20first%20component.md), we made a `Profile` component and a `Gallery` component. 

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

These currently live in a **root component file**, named `App.js` in this example. 

## Exporting and importing a component

What if you want to change the landing screen in the future and put a list of science books there? Or place all the profiles somewhere else? It makes sense to move `Gallery` and `Profile` out of the root component file. This will make them more modular and reusable in other files. You can move a component in three steps:

1. **Make** a new JS file to put the components in.
2. **Export** your function component from that file (using either [default](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/export#using_the_default_export) or [named](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/export#using_named_exports) exports).
3. **Import** it in the file where you’ll use the component (using the corresponding technique for importing [default](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/import#importing_defaults) or [named](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/import#import_a_single_export_from_a_module) exports).

```JSX
// App.js
import Gallery from './Gallery.js';

export default function App() {
  return (
    <Gallery />
  );
}

// Gallery.js
function Profile() {
  return (
    <img
      src="https://react.dev/images/docs/scientists/QIrZWGIs.jpg"
      alt="Alan L. Hart"
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

_Note: Both `'./Gallery.js'` or `'./Gallery'` will work with React, though the former is closer to how [native ES Modules](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules) work.

### Default vs named exports

There are two primary ways to export values with JS: default and named exports. **A file can have no more than one _default_ export, but it can have as many _named_ exports as you like.**

![](../../../assets/Pasted%20image%2020260729163142.png)

How you export your component dictates how you must import it.

| Syntax  | Export statement                      | Import statement                        |
| ------- | ------------------------------------- | --------------------------------------- |
| Default | `export default function Button() {}` | `import Button from './Button.js';`     |
| Named   | `export function Button() {}`         | `import { Button } from './Button.js';` |
When you write a default import, we can put any name we want after `import`. In contrast, with named imports, the name has to match on both sides. That's why they are called _named_ imports!

**People often use default exports if the file exports only one component, and use named exports if it exports multiple components and values.**

## Exporting and importing multiple components from the same file 

If you want to export multiple components from the same file, simply use _named exports_.

## Recap

On this page you learned:

- What a root component file is
- How to import and export a component
- When and how to use default and named imports and exports
- How to export multiple components from the same file