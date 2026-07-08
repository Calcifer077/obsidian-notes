---
title: Rest parameters and spread syntax
source: https://javascript.info/rest-parameters-spread
created: 2026-07-08
tags:
  - javascript
---
## Rest parameters `...`

A function can be called with any number of arguments, no matter how it is defined.

```js
function sum(a, b) {
	return a + b;
}

alert( sum (1, 2,3,4,5,5));
```

Only the first two parameters will be considered and rest will be ignored. The rest of the parameters can be included by using three dots `...` followed by the name of the array that will contain them.

```js
function sum(...args) {}

alert( sum (1, 2,3))
```

`args` is a array which will contain all the passed parameters.

If you want to specify names for some variables you can do that too.

```js
function showName(firstName, lastName, ...titles){}
```

	The rest parameters must be at the end.

## The "arguments" variables

There is also a special array-like object named `arguments` that contains all arguments by their index.

```js
function showName() {
  alert( arguments.length );
  alert( arguments[0] );
  alert( arguments[1] );

  // it's iterable
  // for(let arg of arguments) alert(arg);
}

// shows: 2, Julius, Caesar
showName("Julius", "Caesar");

// shows: 1, Ilya, undefined (no second argument)
showName("Ilya");
```

`arguments` is an old syntax before rest parameters came along.

`arguments` is both array-like and iterables, its not an array. It doesn't support array methods, so we can't call `arguments.map(...)`. 

### Arrow functions do not have `arguments`
Like `this` keyword, `arguments` will be taken from the outer normal function.

## Spread syntax

Spread syntax does the opposite of rest, it will spread a array into a list of numeric arguments.

```js
let arr = [3, 5, 1];

alert( Math.max(...arr) )
```

You can also use it to merge two or more arrays:

```js
let arr = [3, 5, 1]
let arr2 = [8, 9, 15]

let merged = [0, ...arr, 2, ...arr2];

alert(merged); // 0,3,5,1,2,8,9,15 (0, then arr, then 2, then arr2)
```

We can apply spread syntax to any iterables like a string (will convert it into array of characters) to a array. You could also use `Array.from` for the same purpose.

```js
let str = "Hello";

alert( [...str])
alert( Array.from(str));
```

- `Array.from` works on both array-like and iterables.
- Spread syntax works only with iterables.

Spread operator also works for objects.

We can use the spread operator for copying of arrays and objects. The resulting object will be same (after converting to JSON) but will point to different memory location.

## Summary

When we see `"..."` in the code, it is either rest parameters or the spread syntax.

There’s an easy way to distinguish between them:

- When `...` is at the end of function parameters, it’s “rest parameters” and gathers the rest of the list of arguments into an array.
- When `...` occurs in a function call or alike, it’s called a “spread syntax” and expands an array into a list.

Use patterns:

- Rest parameters are used to create functions that accept any number of arguments.
- The spread syntax is used to pass an array to functions that normally require a list of many arguments.

Together they help to travel between a list and an array of parameters with ease.

All arguments of a function call are also available in “old-style” `arguments`: array-like iterable object.