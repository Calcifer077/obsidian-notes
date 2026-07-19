---
title: Arrow functions
source: https://javascript.info/arrow-functions
created: 2026-07-19
tags:
  - javascript
---
_Note: First part of this note can be found here [Link](https://javascript.info/arrow-functions-basics)_

## Arrow functions have no "this"

Arrow functions do not have `this`. If `this` is accessed, it is taken from the outside.

_Note: By taking from the outside, it means that the outer lexical env and it will go up till `window` until it finds it._

An instance of it to iterate inside an object method.

```js
let group = {
  title: "Our Group",
  students: ["John", "Pete", "Alice"],

  showList() {
    this.students.forEach(
      student => alert(this.title + ': ' + student)
    );
  }
};

group.showList();
```

Here in `forEach`, the arrow function is used, so `this.title` in it is exactly the same as in the outer method `showList`. That is `group.title`.

## Arrows have no "arguments"

Arrow functions also have no `arguments` variable.

That's great for decorators, when we need to forward a call with the current `this` and `arguments`.

For instance, `defer(f, ms)` gets a function and returns a wrapper around it that delays the call by `ms` milliseconds:

```js
function defer(f, ms) {
  return function() {
    setTimeout(() => f.apply(this, arguments), ms);
  };
}

function sayHi(who) {
  alert('Hello, ' + who);
}

let sayHiDeferred = defer(sayHi, 2000);
sayHiDeferred("John"); // Hello, John after 2 seconds
```

The same without an arrow function would look like:

```js
function defer(f, ms) {
  return function(...args) {
    let ctx = this;
    setTimeout(function() {
      return f.apply(ctx, args);
    }, ms);
  };
}
```

Here we had to create additional variables `args` and `ctx` so that the function inside `setTimeout` could take them.

## Summary

Arrow functions:

- Do not have `this`
- Do not have `arguments`
- Can’t be called with `new`
- They also don’t have `super`

That’s because they are meant for short pieces of code that do not have their own “context”, but rather work in the current one. And they really shine in that use case.