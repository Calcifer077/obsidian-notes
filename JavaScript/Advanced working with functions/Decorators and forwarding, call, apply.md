---
title: Decorators and forwarding, call/apply
source: https://javascript.info/call-apply-decorators
created: 2026-07-15
tags:
  - javascript
---
JS gives exceptional flexibility when dealing with functions. They can be passed around, used as objects, and now we'll see how to _forward_ calls between them and _decorate_ them.

## Decorator

Let's say we have a function `slow` which is CPU-heavy, but its results are stable (give same result for same input). It means that we can cache the output of this function call.

But instead of adding that functionality into `slow`, we'll create a wrapper function, that adds caching. 

```js
function slow(x) {
  // there can be a heavy CPU-intensive job here
  alert(`Called with ${x}`);
  return x;
}

function cachingDecorator(func) {
  let cache = new Map();

  return function(x) {
    if (cache.has(x)) {    // if there's such key in cache
      return cache.get(x); // read the result from it
    }

    let result = func(x);  // otherwise call func

    cache.set(x, result);  // and cache (remember) the result
    return result;
  };
}

slow = cachingDecorator(slow);

alert( slow(1) ); // slow(1) is cached and the result returned
alert( "Again: " + slow(1) ); // slow(1) result returned from cache

alert( slow(2) ); // slow(2) is cached and the result returned
alert( "Again: " + slow(2) ); // slow(2) result returned from cache
```

In the code above `cachingDecorator` is a _decorator_: a special function that takes another function and alters its behaviour. We can apply this wrapper to any function. By separating caching from the main function we also keep the main code simpler.

The result of `cachingDecorator(func)` is a “wrapper”: `function(x)` that “wraps” the call of `func(x)` into caching logic:

![](../../assets/Pasted%20image%2020260715152847.png)

Benefits of using a separate `cachingDecorator` instead of altering the code of `slow` itself:

- The `cachingDecorator` is reusable. We can apply it to another function.
- The caching logic is separate, it did not increase the complexity of `slow` itself (if there was any).
- We can combine multiple decorators if needed (other decorators will follow).

## Using "func.call" for the context

The caching decorator mentioned above is not suited to work with object methods. 

For instance, in the code below `worker.slow()` stops working after the decoration:

```js
// we'll make worker.slow caching
let worker = {
  someMethod() {
    return 1;
  },

  slow(x) {
    // scary CPU-heavy task here
    alert("Called with " + x);
    return x * this.someMethod(); // (*)
  }
};

// same code as before
function cachingDecorator(func) {
  let cache = new Map();
  return function(x) {
    if (cache.has(x)) {
      return cache.get(x);
    }
    let result = func(x); // (**)
    cache.set(x, result);
    return result;
  };
}

alert( worker.slow(1) ); // the original method works

worker.slow = cachingDecorator(worker.slow); // now make it caching

alert( worker.slow(2) ); // Whoops! Error: Cannot read property 'someMethod' of undefined
```

The error occurs in the line `(*)` that tries to access `this.someMethod` and fails. The reason is that the wrapper calls the original function as `func(x)` in the line `(**)`, and when called like that, the function gets `this = undefined`. 

Why is it `undefined`?

Because in js, the value of `this` is determined by the object before the dot notation which is absent in the above case. 

We would observer a similar behaviour if we tried to run:

```js
let func = worker.slow;
func(2);
```

So, the wrapper passes the call to the original method, but without the context `this`. Hence the error.

Let's fix it.

There’s a special built-in function method [func.call(context, …args)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call) that allows to call a function explicitly setting `this`.

```js
func.call(contex, arg1, arg2, ...);
```

If runs `func` providing the first argument as `this`, and the next as the arguments. 

To put it simply, below two calls do almost the same:

```js
func(1, 2, 3);
func.call(obj, 1, 2, 3)
```

They both call `func` with arguments `1`, `2` and `3`. The only difference is that `func.call` also sets `this` to `obj`.

Another example:

```js
function sayHi() {
  alert(this.name);
}

let user = { name: "John" };
let admin = { name: "Admin" };

// use call to pass different objects as "this"
sayHi.call( user ); // John
sayHi.call( admin ); // Admin
```

In our case, we can use `call` in the wrapper to pass the context to the original function:

```js
let worker = {
  someMethod() {
    return 1;
  },

  slow(x) {
    alert("Called with " + x);
    return x * this.someMethod(); // (*)
  }
};

function cachingDecorator(func) {
  let cache = new Map();
  return function(x) {
    if (cache.has(x)) {
      return cache.get(x);
    }
    let result = func.call(this, x); // "this" is passed correctly now
    cache.set(x, result);
    return result;
  };
}

worker.slow = cachingDecorator(worker.slow); // now make it caching

alert( worker.slow(2) ); // works
alert( worker.slow(2) ); // works, doesn't call the original (cached)
```

How does `this` get value of `worker`?

We do `worker.slow(2)` and the value of `this` inside the function is set to `worker` which we passed further down the line.

## Going multi-argument

Say our `slow` function now needs two arguments `min` and `max`.

```js
let worker = {
  slow(min, max) {
    return min + max; // scary CPU-hogger is assumed
  }
};

// should remember same-argument calls
worker.slow = cachingDecorator(worker.slow);
```

How can we cache it?

The best way would be to use a string of `min + ', ' + max` as a key in the map.

Another problem is that we need to pass all arguments inside decorator to `func.call`. We can do so using `arguments` object instead of manually writing each argument.

```js
let worker = {
  slow(min, max) {
    alert(`Called with ${min},${max}`);
    return min + max;
  }
};

function cachingDecorator(func, hash) {
  let cache = new Map();
  return function() {
    let key = hash(arguments); // (*)
    if (cache.has(key)) {
      return cache.get(key);
    }

    let result = func.call(this, ...arguments); // (**)

    cache.set(key, result);
    return result;
  };
}

function hash(args) {
  return args[0] + ',' + args[1];
}

worker.slow = cachingDecorator(worker.slow, hash);

alert( worker.slow(3, 5) ); // works
alert( "Again " + worker.slow(3, 5) ); // same (cached)
```

Now the decorator works for any number of arguments (not the hash function). 

There are two changes:

- In the line `(*)` it calls `hash` to create a single key from `arguments`. Here we use a simple “joining” function that turns arguments `(3, 5)` into the key `"3,5"`. More complex cases may require other hashing functions.
- Then `(**)` uses `func.call(this, ...arguments)` to pass both the context and all arguments the wrapper got (not just the first one) to the original function.

## func.apply

Instead of `func.call(this, ...arguments)` we could use `func.apply(this, arguments)`.

The syntax is:

```js
func.apply(context, args)
```

It runs the `func` setting `this=context` and using an array-like object `args` as the list of arguments.

The only syntax difference between `call` and `apply` is that `call` expects a list of arguments, while `apply` takes an array-like object with them.

So these two calls are almost equivalent:

```js
func.call(context, ...args);
func.apply(context, args);
```

They perform the same call of `func` with given context and arguments.

There's only a subtle difference regarding `args`:

- The spread syntax `...` allows to pass _iterable_ `args` as the list to `call`.
- The `apply` accepts only _array-like_ `args`.

…And for objects that are both iterable and array-like, such as a real array, we can use any of them, but `apply` will be faster, because most JavaScript engines internally optimize it better.

Passing all arguments along with the context to another function is called _call forwarding_.

## Borrowing a method

Let's try to make our hash function universal (work for any number of arguments).

We can do something like this:

```js
function hash(args) {
  return args.join();
}
```

This won't work because we are calling `hash(arguments)`, and `arguments` object is both iterable and array-like, but not a real array.

There's an easy way to use array join:

```js
function hash() {
  alert( [].join.call(arguments) ); // 1,2
}

hash(1, 2);
```

This trick is called _method borrowing_.

We take (borrow) a join method from a regular array (`[].join`) and use `[].join.call` to run it in the context of `arguments`.

Refer to source. pls.

## Decorators and function properties

It is usually safe to use decorators except when the original function had properties associated to it. If the original function has properties on it, like `func.calledCount`, then the decorator one will not provide them. 

## Summary

_Decorator_ is a wrapper around a function that alters its behavior. The main job is still carried out by the function.

Decorators can be seen as “features” or “aspects” that can be added to a function. We can add one or add many. And all this without changing its code!

To implement `cachingDecorator`, we studied methods:

- [func.call(context, arg1, arg2…)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call) – calls `func` with given context and arguments.
- [func.apply(context, args)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply) – calls `func` passing `context` as `this` and array-like `args` into a list of arguments.

The generic _call forwarding_ is usually done with `apply`:

```javascript
let wrapper = function() {
  return original.apply(this, arguments);
};
```

We also saw an example of _method borrowing_ when we take a method from an object and `call` it in the context of another object. It is quite common to take array methods and apply them to `arguments`. The alternative is to use rest parameters object that is a real array.

## Further reading

- [Iterables](../Data%20types/Iterables.md)
- [Function object, NFE](Function%20object,%20NFE.md)