---
title: Async/await
source: https://javascript.info/async-await
author:
published:
created: 2026-06-05
description: In this note you will learn about how to about async and await.
tags:
  - javascript
---
There’s a special syntax to work with promises in a more comfortable fashion, called “async/await”. It’s surprisingly easy to understand and use.

## Async functions

Let’s start with the `async` keyword. It can be placed before a function, like this:

```javascript
async function f() {
  return 1;
}
```

The word “async” before a function means one simple thing: a function always returns a promise. Other values are wrapped in a resolved promise automatically.

For instance, this function returns a resolved promise with the result of `1`; let’s test it:

```javascript
async function f() {
  return 1;
}

f().then(alert); // 1
```

…We could explicitly return a promise, which would be the same:

```javascript
async function f() {
  return Promise.resolve(1);
}

f().then(alert); // 1
```

So, `async` ensures that the function returns a promise, and wraps non-promises in it. Simple enough, right? But not only that. There’s another keyword, `await`, that works only inside `async` functions, and it’s pretty cool.

## Await

The syntax:

```javascript
// works only inside async functions
let value = await promise;
```

The keyword `await` makes JavaScript wait until that promise settles and returns its result.

Here’s an example with a promise that resolves in 1 second:

```javascript
async function f() {

  let promise = new Promise((resolve, reject) => {
    setTimeout(() => resolve("done!"), 1000)
  });

  let result = await promise; // wait until the promise resolves (*)

  alert(result); // "done!"
}

f();
```

The function execution “pauses” at the line `(*)` and resumes when the promise settles, with `result` becoming its result. So the code above shows “done!” in one second.

Let’s emphasize: `await` literally suspends the function execution until the promise settles, and then resumes it with the promise result. That doesn’t cost any CPU resources, because the JavaScript engine can do other jobs in the meantime: execute other scripts, handle events, etc.

It’s just a more elegant syntax of getting the promise result than `promise.then`. And, it’s easier to read and write.

### Can't use `await` in regular functions
If we try to use `await` in a non-async function, there would be a syntax error:

```javascript
function f() {
  let promise = Promise.resolve(1);
  let result = await promise; // Syntax error
}
```

We may get this error if we forget to put `async` before a function. As stated earlier, `await` only works inside an `async` function.

Let’s take the `showAvatar()` example from the chapter [Promises chaining](Promises%20chaining.md) and rewrite it using `async/await`:

1. We’ll need to replace `.then` calls with `await`.
2. Also we should make the function `async` for them to work.

```javascript
async function showAvatar() {

  // read our JSON
  let response = await fetch('/article/promise-chaining/user.json');
  let user = await response.json();

  // read github user
  let githubResponse = await fetch(`https://api.github.com/users/${user.name}`);
  let githubUser = await githubResponse.json();

  // show the avatar
  let img = document.createElement('img');
  img.src = githubUser.avatar_url;
  img.className = "promise-avatar-example";
  document.body.append(img);

  // wait 3 seconds
  await new Promise((resolve, reject) => setTimeout(resolve, 3000));

  img.remove();

  return githubUser;
}

showAvatar();
```

Pretty clean and easy to read, right? Much better than before.
### Modern browsers allow top-level `await` in modules
In modern browsers, `await` on top level works just fine, when we’re inside a module. 

For instance:

```javascript
// we assume this code runs at top level, inside a module
let response = await fetch('/article/promise-chaining/user.json');
let user = await response.json();

console.log(user);
```

If we’re not using modules, or [older browsers](https://caniuse.com/mdn-javascript_operators_await_top_level) must be supported, there’s a universal recipe: wrapping into an anonymous async function.

Like this:

```javascript
(async () => {
  let response = await fetch('/article/promise-chaining/user.json');
  let user = await response.json();
  ...
})();
```

### `await` accepts “thenables”
Like `promise.then`, `await` allows us to use thenable objects (those with a callable `then` method). The idea is that a third-party object may not be a promise, but promise-compatible: if it supports `.then`, that’s enough to use it with `await`.

Here’s a demo `Thenable` class; the `await` below accepts its instances:

```javascript
class Thenable {
  constructor(num) {
    this.num = num;
  }
  then(resolve, reject) {
    alert(resolve);
    // resolve with this.num*2 after 1000ms
    setTimeout(() => resolve(this.num * 2), 1000); // (*)
  }
}

async function f() {
  // waits for 1 second, then result becomes 2
  let result = await new Thenable(1);
  alert(result);
}

f();
```

If `await` gets a non-promise object with `.then`, it calls that method providing the built-in functions `resolve` and `reject` as arguments (just as it does for a regular `Promise` executor). Then `await` waits until one of them is called (in the example above it happens in the line `(*)`) and then proceeds with the result.

### Async class methods
To declare an async class method, just prepend it with `async`:

```javascript
class Waiter {
  async wait() {
    return await Promise.resolve(1);
  }
}

new Waiter()
  .wait()
  .then(alert); // 1 (this is the same as (result => alert(result)))
```

The meaning is the same: it ensures that the returned value is a promise and enables `await`.

## Error handling

If a promise resolves normally, then `await promise` returns the result. But in the case of a rejection, it throws the error, just as if there were a `throw` statement at that line.

This code:

```javascript
async function f() {
  await Promise.reject(new Error("Whoops!"));
}
```

…is the same as this:

```javascript
async function f() {
  throw new Error("Whoops!");
}
```

In real situations, the promise may take some time before it rejects. In that case there will be a delay before `await` throws an error.

We can catch that error using `try..catch`, the same way as a regular `throw`:

```javascript
async function f() {

  try {
    let response = await fetch('http://no-such-url');
  } catch(err) {
    alert(err); // TypeError: failed to fetch
  }
}

f();
```

In the case of an error, the control jumps to the `catch` block. We can also wrap multiple lines:

```javascript
async function f() {

  try {
    let response = await fetch('/no-user-here');
    let user = await response.json();
  } catch(err) {
    // catches errors both in fetch and response.json
    alert(err);
  }
}

f();
```

If we don’t have `try..catch`, then the promise generated by the call of the async function `f()` becomes rejected. We can append `.catch` to handle it:

```javascript
async function f() {
  let response = await fetch('http://no-such-url');
}

// f() becomes a rejected promise
f().catch(alert); // TypeError: failed to fetch // (*)
```

If we forget to add `.catch` there, then we get an unhandled promise error (viewable in the console). We can catch such errors using a global `unhandledrejection` event handler as described in the chapter [Error handling with promises](Error%20handling%20with%20promises.md).

### `async/await` and `promise.then/catch`
When we use `async/await`, we rarely need `.then`, because `await` handles the waiting for us. And we can use a regular `try..catch` instead of `.catch`. That’s usually (but not always) more convenient.

But at the top level of the code, when we’re outside any `async` function, we’re syntactically unable to use `await`, so it’s a normal practice to add `.then/catch` to handle the final result or falling-through error, like in the line `(*)` of the example above.

### `async/await` works well with `Promise.all`
When we need to wait for multiple promises, we can wrap them in `Promise.all` and then `await`:

```javascript
// wait for the array of results
let results = await Promise.all([
  fetch(url1),
  fetch(url2),
  ...
]);
```

In the case of an error, it propagates as usual, from the failed promise to `Promise.all`, and then becomes an exception that we can catch using `try..catch` around the call.

## Summary

The `async` keyword before a function has two effects:

1. Makes it always return a promise.
2. Allows `await` to be used in it.

The `await` keyword before a promise makes JavaScript wait until that promise settles, and then:

1. If it’s an error, an exception is generated — same as if `throw error` were called at that very place.
2. Otherwise, it returns the result.

Together they provide a great framework to write asynchronous code that is easy to both read and write.

With `async/await` we rarely need to write `promise.then/catch`, but we still shouldn’t forget that they are based on promises, because sometimes (e.g. in the outermost scope) we have to use these methods. Also `Promise.all` is nice when we are waiting for many tasks simultaneously.

## Questions

### Q. Call async from non-async
We have a "regular" function called `f`. How can you call the `async` function `wait()` and use its result inside of `f`?
```js
async function wait() {
  await new Promise(resolve => setTimeout(resolve, 1000));

  return 10;
}

function f() {
  // ...what should you write here?
  // we need to call async wait() and wait to get 10
  // remember, we can't use "await"
}
```
### Ans:
Just tread `async` call as promise and attack `.then` to it:
```js
async function wait() {
  await new Promise(resolve => setTimeout(resolve, 1000));

  return 10;
}

function f() {
  // shows 10 after 1 second
  wait().then(result => alert(result));
}

f();
```

---
### Q. Dangerous Promise.all
Let's say we have a connection to a remote service, such as a database. There's two functions: `connect()` and `disconnet()`.

When connected, we can send requests using `database.query(...)` – an async function which usually returns the result but also may throw an error.

Here’s a simple implementation:
```js
let database;

function connect() {
  database = {
    async query(isOk) {
      if (!isOk) throw new Error('Query failed');
    }
  };
}

function disconnect() {
  database = null;
}

// intended usage:
// connect()
// ...
// database.query(true) to emulate a successful call
// database.query(false) to emulate a failed call
// ...
// disconnect()
```

Now here's the problem. We wrote the code to connect and send 3 queries in parallel (all of them take different time, e.g. 100, 200 and 300ms), then disconnect:
```js
// Helper function to call async function `fn` after `ms` milliseconds
function delay(fn, ms) {
  return new Promise((resolve, reject) => {
    setTimeout(() => fn().then(resolve, reject), ms);
  });
}

async function run() {
  connect();

  try {
    await Promise.all([
      // these 3 parallel jobs take different time: 100, 200 and 300 ms
      // we use the `delay` helper to achieve this effect
      delay(() => database.query(true), 100),
      delay(() => database.query(false), 200),
      delay(() => database.query(false), 300)
    ]);
  } catch(error) {
    console.log('Error handled (or was it?)');
  }

  disconnect();
}

run();
```

Two of these queries happen to be unsuccessful, but we’re smart enough to wrap the `Promise.all` call into a `try..catch` block.

However, this doesn’t help! This script actually leads to an uncaught error in console!

Why? How to avoid it?

### Ans:
Before answer, some things about the above code. How does the code get access to `database`. `database` is a global variable, and when we call `connect` in `run`, it is initialized with query function. Due to which we can access it all over the code, until we `disconnect` it which marks it as `null`.

The root of the problem is that `Promise.all` immediately rejects when one of its promises rejects, but it do nothing to cancel other promises. 

In our case, the second query fails, so `Promise.all` rejects, and the `try...catch` block catches this error. Meanwhile, other promises are _not affected_ - they continue their execution. In our case, the third query throws an error of its own after a bit of time, And that error is never caught, we can see it in the console. The problem is especially dangerous in server-side environments, such as Node.js, when an uncaught error may cause the process to crash.

An ideal solution would be to cancel all unfinished queries when one of them fails. This way we avoid any potential errors.

However, the bad news is that service calls (such as `database.query`) are often implemented by a 3rd-party library which doesn't support cancellation. Then there's no way to cancel a call.

#### Solutions
**Solution 1: Custom `Promise.all` that ignores late errors.**
```js
function customPromiseAll(promises) {
  return new Promise((resolve, reject) => {
    const results = [];
    let resultsCount = 0;
    let hasError = false; // we'll set it to true upon first error

    promises.forEach((promise, index) => {
      promise
        .then(result => {
          if (hasError) return; // ignore the promise if already errored
          results[index] = result;
          resultsCount++;
          if (resultsCount === promises.length) {
            resolve(results); // when all results are ready - successs
          }
        })
        .catch(error => {
          if (hasError) return; // ignore the promise if already errored
          hasError = true; // wops, error!
          reject(error); // fail with rejection
        });
    });
  });
}
```
This prevents the uncaught error, but has a trade-off - remaining promises are still running, you're just silently ignoring them. If a still-running query does something important (like a DB write), you've abandoned it.

**Solution 2: Wait for ALL promises to settle before rejecting**
```js
function customPromiseAllWait(promises) {
  return new Promise((resolve, reject) => {
    const results = new Array(promises.length);
    let settledCount = 0;
    let firstError = null;

    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(result => {
          results[index] = result;
        })
        .catch(error => {
          if (firstError === null) {
            firstError = error;
          }
        })
        .finally(() => {
          settledCount++;
          if (settledCount === promises.length) {
            if (firstError !== null) {
              reject(firstError);
            } else {
              resolve(results);
            }
          }
        });
    });
  });
}
```
This is safer - it waits until all 3 queries are done before resolving/rejecting. So `disconnect()` won't run prematurely. The downside is you still only surface the _first_ error, silently dropping the rest.

**Solution 3 – `Promise.allSettled` + `AggregateError`**
```js
// wait for all promises to settle
// return results if no errors
// throw AggregateError with all errors if any
function allOrAggregateError(promises) {
  return Promise.allSettled(promises).then(results => {
    const errors = [];
    const values = [];

    results.forEach((res, i) => {
      if (res.status === 'fulfilled') {
        values[i] = res.value;
      } else {
        errors.push(res.reason);
      }
    });

    if (errors.length > 0) {
      throw new AggregateError(errors, 'One or more promises failed');
    }

    return values;
  });
}
```
`Promise.allSettled` never short-circuits - it always waits for every promise to finish. Then you collect all errors and bundle them into a single `AggregateError`. This is the most thorough approach: nothing is dropped, nothing runs unhandled.

**How to use any of them?**
Before:
```js
await Promise.all([query1, query2, query3]);
```
After:
```js
await customPromiseAllWait([query1, query2, query3]);
// or
await allOrAggregateError([query1, query2, query3]);
```



---