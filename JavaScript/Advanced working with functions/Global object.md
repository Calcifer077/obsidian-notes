---
title: Global object
source: https://javascript.info/global-object
created: 2026-07-09
tags:
  - javascript
---
The global object provides variables and functions that are available anywhere. In a browser it is named `window`, for Node.js it is `global`.

All properties of the global object can be accessed directly:

```js
alert("Hello");
// is the same as
window.alert("Hello");
```

In a browser, global functions declarations (not function expressions) and variables declared with `var` (not `let`/`const`) become the property of the global object.

```js
var gVar = 5;

alert(window.gVar); // 5 (became a property of the global object)
```

If we try to do this for `let`, it will give us `undefined`. 

## Using for polyfills

We can use the global object to test for support of modern language features and can use the absence to polyfill those features.

```js
if (!window.Promise) {
  window.Promise = ... // custom implementation of the modern language feature
}
```

## Summary

- The global object holds variables that should be available everywhere.
    
    That includes JavaScript built-ins, such as `Array` and environment-specific values, such as `window.innerHeight` – the window height in the browser.
    
- The global object has a universal name `globalThis`.
    
    …But more often is referred by “old-school” environment-specific names, such as `window` (browser) and `global` (Node.js).
    
- We should store values in the global object only if they’re truly global for our project. And keep their number at minimum.
    
- In-browser, unless we’re using [modules](https://javascript.info/modules), global functions and variables declared with `var` become a property of the global object.
    
- To make our code future-proof and easier to understand, we should access properties of the global object directly, as `window.x`.

