---
title: Function object, NFE
source: https://javascript.info/function-object
created: 2026-07-09
tags:
  - javascript
---
In JS, functions are objects. A good way to imagine functions is as callable "action objects". We can not only call them, but also treat them as objects: add/remove properties, pass by reference etc.

## The "name" property

A function's name is accessible as the "name" property.

```js
function sayHi() {
  alert("Hi");
}

alert(sayHi.name); // sayHi

// ---------------------------------------------------------

let sayHi = function() {
  alert("Hi");
};

alert(sayHi.name); // sayHi (there's a name!)

// ----------------------------------------------------------

function f(sayHi = function() {}) {
  alert(sayHi.name); // sayHi (works!)
}

f();
```

In the specification, this feature is called a "contextual name". 

Object methods have names too:

```js
let user = {

  sayHi() {
    // ...
  },

  sayBye: function() {
    // ...
  }

}

alert(user.sayHi.name); // sayHi
alert(user.sayBye.name); // sayBye
```

When JS engine can't figure out the name it will be a `<empty string>`.

```js
// function created inside array
let arr = [function() {}];

alert( arr[0].name ); // <empty string>
// the engine has no way to set up the right name, so there is none
```

## The "length" property

`function.length` return the number of function parameters. It ignores rest parameters. 

## Custom properties

We can also add properties of our own.

```js
function sayHi() {
  alert("Hi");

  // let's count how many times we run
  sayHi.counter++;
}
sayHi.counter = 0; // initial value

sayHi(); // Hi
sayHi(); // Hi

alert( `Called ${sayHi.counter} times` ); // Called 2 times
```

A property assigned to a function like `sayHi.counter = 0` does not define a local variable `counter` inside it. In other words, a property `counter` and a variable `let counter` are two unrelated things and won't interfere with each other.

We can also implement closures using function properties.

```js
function makeCounter() {
  // instead of:
  // let count = 0

  function counter() {
    return counter.count++;
  };

  counter.count = 0;

  return counter;
}

let counter = makeCounter();
alert( counter() ); // 0
alert( counter() ); // 1
```

One thing to keep in mind is that in case of closures using function properties, outer code can also make changes to variables defined in the so called _lexical env_ of the inner variable which is not possible in traditional way.

## Named function expression

Named function expression, or NFE, is a term for function expressions that have a name.

```js
let sayHi = function func(who) {
  alert(`Hello, ${who}`);
};
```

`func` is the name of above function expression.

First let’s note, that we still have a Function Expression. Adding the name `"func"` after `function` did not make it a Function Declaration, because it is still created as a part of an assignment expression. You can still call it using `sayHi()`. `func()` won't work.

There are two special things about naming a function expression:
1. It allows the function to reference itself internally.
2. It is not visible outside the function.

Here's a example:

```js
let sayHi = function func(who) {
  if (who) {
    alert(`Hello, ${who}`);
  } else {
    func("Guest"); // use func to re-call itself
  }
};

sayHi(); // Hello, Guest
```

Above we could have also called `sayHi` in the internal call and it would have worked except for one case.

```js
let sayHi = function(who) {
  if (who) {
    alert(`Hello, ${who}`);
  } else {
    sayHi("Guest"); // Error: sayHi is not a function
  }
};

let welcome = sayHi;
sayHi = null;

welcome(); // Error, the nested sayHi call doesn't work any more!
```

Because when we `welcome()`, the function internally tries to `null("Guest")` (because `sayHi` is `null`) which causes above error. But if we would have using function name it would have worked.

```js
let sayHi = function func(who) {
  if (who) {
    alert(`Hello, ${who}`);
  } else {
    func("Guest"); // Now all fine
  }
};

let welcome = sayHi;
sayHi = null;

welcome(); // Hello, Guest (nested call works)
```

## Summary

Functions are objects.

Here we covered their properties:

- `name` – the function name. Usually taken from the function definition, but if there’s none, JavaScript tries to guess it from the context (e.g. an assignment).
- `length` – the number of arguments in the function definition. Rest parameters are not counted.

If the function is declared as a Function Expression (not in the main code flow), and it carries the name, then it is called a Named Function Expression. The name can be used inside to reference itself, for recursive calls or such.