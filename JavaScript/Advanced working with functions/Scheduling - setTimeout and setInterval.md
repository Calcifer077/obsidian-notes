---
title: "Scheduling: setTimeout and setInterval"
source: https://javascript.info/settimeout-setinterval
created: 2026-07-14
tags:
  - javascript
---
We may decide to execute a function not right now, but at a certain time later.

There are two methods for it:
- `setTimeout` allows us to run a function once after the interval of time.
- `setInterval` allows us to run a function repeatedly, starting after the interval of time, then repeating continuously at that interval. 

_Note: These methods are not a part of JS specification. But most environments support them._

## `setTimeout`

```js
let timerId = setTimeout(func|code, [delay], [arg1], [arg2], ...)
```

Parameters:

`func|code`: Function or a string of code to execute. Usually, that’s a function. For historical reasons, a string of code can be passed, but that’s not recommended. It will execute but will also give a error.

`delay`: The delay before run, in milliseconds. By default it is 0.

`arg1`, `arg2` :  Arguments for the function.

Example:

```js
function sayHi() {
  alert('Hello');
}

setTimeout(sayHi, 1000);

// With arguments
function sayHi(phrase, who) {
  alert( phrase + ', ' + who );
}

setTimeout(sayHi, 1000, "Hello", "John"); // Hello, John
```

## Cancelling with `clearTimeout`

A call to `setTimeout` returns a "timer identifier" `timerId` that we can use to cancel the execution.

```js
let timerId = setTimeout(...);
clearTimeout(timerId);
```

## `setInterval`

```js
let timerId = setInterval(func|code, [delay], [arg1], [arg2], ...)
```

All arguments have the same meaning. But unlike `setTimeout` it runs the function not only once, but regularly after the given interval of time.

To stop further calls, we should call `clearInterval(timerId)`.

```js
// repeat with the interval of 2 seconds
let timerId = setInterval(() => alert('tick'), 2000);

// after 5 seconds stop
setTimeout(() => { clearInterval(timerId); alert('stop'); }, 5000);
```

_Note: Time goes on even if we don't cancel the alert._

## Nested `setTimeout`

There are two ways of running something regularly. One is `setInterval` and other is a nested `setTimeout` like this:

```js
/** instead of:
let timerId = setInterval(() => alert('tick'), 2000);
*/

let timerId = setTimeout(function tick() {
  alert('tick');
  timerId = setTimeout(tick, 2000); // (*)
}, 2000);
```

The `setTimeout` above schedules the next call right at the end of the current one `(*)`. 

The nested `setTimeout` is more flexible method than `setInterval`. This way we can modify the next call depending on the result of the current one.

**Nested `setTimeout` allows to set the delay between the executions more precisely than `setInterval`**.

Let's compare both:

`setInterval`

```js
let i = 1;
setInterval(function() {
   func(i++);
}, 100);
```

`setTimeout`:

```js
let i = 1;
setTimeout(function run() {
  func(i++);
  setTimeout(run, 100);
}, 100);
```

For `setInterval` the internal schedular will run `func(i++)` every 100ms:

![](../../assets/Pasted%20image%2020260714215042.png)

Above diagram means that `setInterval` have scheduled `func` to run after every 100ms. Say the first call of `func` takes 10ms, second call will be executed after 90ms, which is not the case for `setTimeout`. No matter how much time is taken by first execution of `func` second execution will only start after 100ms.

![](../../assets/Pasted%20image%2020260714215430.png)

**The nested `setTimeout` ensures a minimum delay (100ms here) between the end of one call and the beginning of the subsequent one.**

#### Garbage collection and `setInterval`/`setTimeout` callback

When a function is passed in `setInterval/setTimeout`, an internal reference is created to it and saved in the scheduler. It prevents the function from being garbage collected, even if there are no other references to it. `setInterval` remains in the memory until `clearInterval` is called. 

There’s a side effect. A function references the outer lexical environment, so, while it lives, outer variables live too. They may take much more memory than the function itself. So when we don’t need the scheduled function anymore, it’s better to cancel it, even if it’s very small.

#### Zero delay is in fact not zero (in a browser)

If you pass 0 to delay, or don't pass anything it all (0 default value), specification states that "after five nested timers, the interval is forced to be at least 4ms".

Example:

```js
let start = Date.now();
let times = [];

setTimeout(function run() {
  times.push(Date.now() - start); // remember delay from the previous call

  if (start + 100 < Date.now()) alert(times); // show the delays after 100ms
  else setTimeout(run); // else re-schedule
});

// an example of the output:
// 1,1,1,1,9,15,20,24,30,35,40,45,50,55,59,64,70,75,80,85,90,95,100
```

The similar thing happens if we use `setInterval` instead of `setTimeout`: `setInterval(f)` runs `f` few times with zero-delay, and afterwards with 4+ ms delay.

This is only browser specific, and it doesn't exist in server side js.

## Summary

- Methods `setTimeout(func, delay, ...args)` and `setInterval(func, delay, ...args)` allow us to run the `func` once/regularly after `delay` milliseconds.
- To cancel the execution, we should call `clearTimeout/clearInterval` with the value returned by `setTimeout/setInterval`.
- Nested `setTimeout` calls are a more flexible alternative to `setInterval`, allowing us to set the time _between_ executions more precisely.
- Zero delay scheduling with `setTimeout(func, 0)` (the same as `setTimeout(func)`) is used to schedule the call “as soon as possible, but after the current script is complete”.
- The browser limits the minimal delay for five or more nested calls of `setTimeout` or for `setInterval` (after 5th call) to 4ms. That’s for historical reasons.

Please note that all scheduling methods do not _guarantee_ the exact delay.

For example, the in-browser timer may slow down for a lot of reasons:

- The CPU is overloaded.
- The browser tab is in the background mode.
- The laptop is on battery saving mode.

All that may increase the minimal timer resolution (the minimal delay) to 300ms or even 1000ms depending on the browser and OS-level performance settings.