---
title: The "new function" syntax
source: https://javascript.info/new-function
created: 2026-07-10
tags:
  - javascript
---
## Syntax

```js
let func = new Function ([arg1, arg2, ...argN], functionBody);
```

The function is created with arguments `arg1...argN` and the given `functionBody`.

Example with two arguments:

```js
let sum = new Function('a', 'b', 'return a + b');

alert( sum(1, 2) ); // 3
```

Example with no argument and a single body:

```js
let sayHi = new Function('alert("Hello")');

sayHi(); // Hello
```

The major difference is that the function is created literally from a string, that is passed at run time. One use case could be when you have to run script inputted by the user. 

## Closure 

Usually, a function remembers where it was born in the special property `[[Enviornment]]`. It references the lexical environment from where it's created.

But when a function is created using `new Function`, its `[[Environment]]` is set to reference not the current lexical env, but the global one. 

```js
function getFunc() {
  let value = "test";

  let func = new Function('alert(value)');

  return func;
}

getFunc()(); // error: value is not defined
```

Such functions doesn't have access to outer variables, only to the global ones.

_Note: Here global variables means the true global lexical env. If you were to move `let value` out of `getFunc()`, `new Function` syntax still wouldn't be able to recognize it. As the browser, node runs scripts as modules even top level variables are not considered as global._

This feature of `new Function` looks strange and doesn't fit into normal conditions, but it is very useful in practice. 

Imagine that we are creating a function from a string. The string will be received from some server or another source (_not known at the time of writing the script_). We can't use regular functions here and have to use the `new Function` syntax. 

Our new function needs to interact with the main script.

What if it could access the outer variables?

The problem is hat before JS is published to production, it's compressed using a _minifier_ - a special program that shrinks code by removing extra comments, spaces, renaming local variables into shorter ones.

For instance, if a function has `let userName`, minifier replaces it with `let a` (or another letter if this one is occupied), and does it everywhere. That’s usually a safe thing to do, because the variable is local, nothing outside the function can access it. And inside the function, minifier replaces every mention of it.

So if `new Function` had access to outer variables, it would be unable to find renamed `userName`.

**If `new Function` had access to outer variables, it would have problems with minifiers.**

To pass something to a function, created as `new Function`, we should use its arguments. 

## Summary 

The syntax:

```javascript
let func = new Function ([arg1, arg2, ...argN], functionBody);
```

For historical reasons, arguments can also be given as a comma-separated list.

These three declarations mean the same:

```javascript
new Function('a', 'b', 'return a + b'); // basic syntax
new Function('a,b', 'return a + b'); // comma-separated
new Function('a , b', 'return a + b'); // comma-separated with spaces
```

Functions created with `new Function`, have `[[Environment]]` referencing the global Lexical Environment, not the outer one. Hence, they cannot use outer variables. But that’s actually good, because it insures us from errors. Passing parameters explicitly is a much better method architecturally and causes no problems with minifiers.