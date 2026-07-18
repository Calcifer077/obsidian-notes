---
title: Function Binding
source: https://javascript.info/bind
created: 2026-07-18
tags:
  - javascript
---
When passing object methods as callbacks, for instance to `setTimeout`, there's a known problem: "losing `this`".

## Losing "this"

Let's take a example:

```js
let user = {
  firstName: "John",
  sayHi() {
    alert(`Hello, ${this.firstName}!`);
  }
};

setTimeout(user.sayHi, 1000); // Hello, undefined!
```

The output above will be `undefined` and not `John`. That's because `setTimeout` got the function `user.sayHi`, separately from the object. The last can be rewritten as:

```js
let f = user.sayHi;
setTimeout(f, 1000); // lost user context
```

In browser, `this` will be set to `window` after being lost (in nodejs to `Timer` object).

Here's how to resolve it:

### Solution 1: a wrapper

The simple solution is to use a wrapping function:

```js
let user = {
  firstName: "John",
  sayHi() {
    alert(`Hello, ${this.firstName}!`);
  }
};

setTimeout(function() {
  user.sayHi(); // Hello, John!
}, 1000);
```

_Note: `this` is determined during runtime. `this` basically becomes the object before `.`. In the problematic case there was no `.` before the function call but it is present in the solution, which resolve the problem (to some extent)._

Now it works, because it receives `user` from the outer lexical environment, and then calls the method normally. 

It looks fine, but there is a slight problem with this. What if before `setTimeout` triggers, `user` changes values? Then it will call the wrong object.

```js
let user = {
  firstName: "John",
  sayHi() {
    alert(`Hello, ${this.firstName}!`);
  }
};

setTimeout(() => user.sayHi(), 1000);

// ...the value of user changes within 1 second
user = {
  sayHi() { alert("Another user in setTimeout!"); }
};

// Another user in setTimeout!
```

### Solution 2: bind

Functions provide a built-in method [bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind) that allows to fix `this`.

The basic syntax is:

```js
let boundFunc = func.bind(context);
```

The result of `func.bind(context)` is a special function-like "exotic object", that is callable as function and transparently passes the call to `func` setting `this=context`.

In other words, calling `boundFunc` is like `func` with fixed `this`. 

Here's a example with arguments.

```js
let user = {
  firstName: "John"
};

function func(phrase) {
  alert(phrase + ', ' + this.firstName);
}

// bind this to user
let funcUser = func.bind(user);

funcUser("Hello"); // Hello, John (argument "Hello" is passed, and this=user)
```

You can even do it for a object method and it will even work if the object changed in future:

```js
let user = {
  firstName: "John",
  sayHi() {
    alert(`Hello, ${this.firstName}!`);
  }
};

let sayHi = user.sayHi.bind(user); // (*)

// can run it without an object
sayHi(); // Hello, John!

setTimeout(sayHi, 1000); // Hello, John!

// even if the value of user changes within 1 second
// sayHi uses the pre-bound value which is reference to the old user object
user = {
  sayHi() { alert("Another user in setTimeout!"); }
};
```

## Partial functions

We can bind not only `this`, but also arguments.

```js
let bound = func.bind(context, [arg1], [arg2], ...);
```

For instance, we have a multiplication function `mul(a, b)`:

```js
function mul(a, b){
	return a * b;
}
```

Let's use `bind` to create a function `double` on its base:

```js
function mul(a, b) {
  return a * b;
}

let double = mul.bind(null, 2);

alert( double(3) ); // = mul(2, 3) = 6
alert( double(4) ); // = mul(2, 4) = 8
alert( double(5) ); // = mul(2, 5) = 10
```

The call to `mul.bind(null, 2)` creates a new function `double` that passes calls to `mul`, fixing `null` as the context and `2` as the first argument. Further arguments are passed "as is".

That’s called [partial function application](https://en.wikipedia.org/wiki/Partial_application) – we create a new function by fixing some parameters of the existing one.

Syntax of `bind` requires us to pass `this` even if we don't use it.

## Going partial without context

What if we'd like to fix some arguments, but not the context `this`?

The native `bind` does not allow that. 

We can implement that on our own:

```js
function partial(func, ...argsBound) {
  return function(...args) { // (*)
    return func.call(this, ...argsBound, ...args);
  }
}

// Usage:
let user = {
  firstName: "John",
  say(time, phrase) {
    alert(`[${time}] ${this.firstName}: ${phrase}!`);
  }
};

// add a partial method with fixed time
user.sayNow = partial(user.say, new Date().getHours() + ':' + new Date().getMinutes());

user.sayNow("Hello");
// Something like:
// [10:00] John: Hello!
```

The result of `partial(func[, arg1, arg2...])` call is a wrapper `(*)` that calls `func` with:

- Same `this` as it gets (for `user.sayNow` call it’s `user`)
- Then gives it `...argsBound` – arguments from the `partial` call (`"10:00"`)
- Then gives it `...args` – arguments given to the wrapper (`"Hello"`)

## Summary

Method `func.bind(context, ...args)` returns a “bound variant” of function `func` that fixes the context `this` and first arguments if given.

Usually we apply `bind` to fix `this` for an object method, so that we can pass it somewhere. For example, to `setTimeout`.

When we fix some arguments of an existing function, the resulting (less universal) function is called _partially applied_ or _partial_.

Partials are convenient when we don’t want to repeat the same argument over and over again. Like if we have a `send(from, to)` function, and `from` should always be the same for our task, we can get a partial and go on with it.