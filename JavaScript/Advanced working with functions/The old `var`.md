---
title: The old "var"
source: https://javascript.info/var
tags:
  - javascript
created: 2026-07-09
---
There are three kinds of variable declarations.
1. `let`
2. `const`
3. `var`

`var` is similar to `let`. _Most of the time_ we can replace `let` by `var` or vice-versa.

## "var" has no block scope

Variables, declared with `var`, are either function-scoped or global-scoped. They are visible through blocks.
`
```js
if (true) {
  var test = true; // use "var" instead of "let"
}

alert(test); // true, the variable lives after if
```

As `var` ignores code blocks, we've got a global variable `test`. If we were to use `let` instead of `var` we would get a `ReferenceError`.

`var` ignores block or loop scope and only follow function scope.

```js
function sayHi() {
  if (true) {
    var phrase = "Hello";
  }

  alert(phrase); // works
}

sayHi();
alert(phrase); // ReferenceError: phrase is not defined
```

## "var" tolerates redeclarations

If we declare the same variable with `let` twice in the same scope, that will give us an error (`SyntaxError`). But redeclarations with `var` works fine, it's value gets reassigned.

## "var" variables can be declared below their use

`var` declarations are processed when the function starts (or script starts for globals). In other words, `var` variables are defined from the beginning of the function, no matter where the definition is (assuming that the definition is not in the nested function).

```js
function sayHi() {
  phrase = "Hello";

  alert(phrase);

  var phrase;
}
sayHi();
```

Above code is technically the same if we were to move `var phrase` to the first line inside function.

Even below will work:

```js
function sayHi() {
  phrase = "Hello"; // (*)

  if (false) {
    var phrase;
  }

  alert(phrase);
}
sayHi();
```

We call such behavior `hoisting` (raising), because all `var` are _hoisted_ (raised) to the top of the function. So in the example above, `if (false)` never executes, but that doesn't matter. The `var` inside it is processed in the beginning of the function, so at the moment of `(*)` the variable exists.

**Declarations are hoisted, but assignments are not.**

```js
function sayHi() {
  var phrase; // declaration works at the start...

  alert(phrase); // undefined

  phrase = "Hello"; // ...assignment - when the execution reaches it.
}

sayHi();
```

## Immediately-invoked function expressions (IIFE)

These are functions which are immediately run once they are created.

```js
(function() {

  var message = "Hello";

  alert(message); // Hello

})();
```

Here, a function expression is created and immediately called. So the code executes right away and has its own private members.

The function expression is wrapped with paranthesis `()`, because JS engine will look at `function` and think that it's a function declaration, which are not allowed to have no name and called immediately after creation.

Function with no name:

```js
// Tries to declare and immediately call a function
function() { // <-- SyntaxError: Function statements require a function name

  var message = "Hello";

  alert(message); // Hello

}();
```

Tried to immediately call a function declaration:

```js
// syntax error because of parentheses below
function go() {

}(); // <-- can't call Function Declaration immediately
```

So, the parentheses around the function is a trick to show JavaScript that the function is created in the context of another expression, and hence it’s a Function Expression: it needs no name and can be called immediately.

Other ways of creating function expression:

```js
(function() {
  alert("Parentheses around the function");
})();

(function() {
  alert("Parentheses around the whole thing");
}());

!function() {
  alert("Bitwise NOT operator starts the expression");
}();

+function() {
  alert("Unary plus starts the expression");
}();
```

## Summary

There are two main differences of `var` compared to `let/const`:

1. `var` variables have no block scope, their visibility is scoped to current function, or global, if declared outside function.
2. `var` declarations are processed at function start (script start for globals).