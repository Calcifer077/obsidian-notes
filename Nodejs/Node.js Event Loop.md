---
title: The Node.js Event Loop
source: https://nodejs.org/learn/asynchronous-work/event-loop-timers-and-nexttick
created: 2026-08-13
tags:
  - nodejs
---
## What is the Event Loop?

The event loop is what allows Node.js to perform non-blocking I/O operations, despite the fact that a single JS thread is used by default.

## Event Loop Explained

When Node.js starts, it initializes the event loop, processes the provided input script which may make async API calls, schedule timers, or call `process.nextTick()`, then begins processing the event loop.

The following diagram shows a simplified overview of the event loop's order of operations.

```
   ┌───────────────────────────┐
   │           timers          │
   └─────────────┬─────────────┘
                 │
                 v
   ┌───────────────────────────┐
┌─>│     pending callbacks     │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │
│  └─────────────┬─────────────┘      ┌───────────────┐
│  ┌─────────────┴─────────────┐      │   incoming:   │
│  │           poll            │<─────┤  connections, │
│  └─────────────┬─────────────┘      │   data, etc.  │
│  ┌─────────────┴─────────────┐      └───────────────┘
│  │           check           │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │      close callbacks      │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤           timers          │
   └───────────────────────────┘
```

> Each box refers to a "phase" of the event loop.

Each phase has a FIFO queue of callbacks to execute. While each phase is special in its own way, generally, when the event loop enters a given phase, it will perform any operations specific to that phase, then execute callbacks in that phase's queue until the queue has been exhausted or the maximum number of callbacks has executed. When the queue has been exhausted or the callback limit is reached, the event loop will move to the next phase, and so on.

Since any of these operations may schedule _more_ operations, and new events processed in the poll phase are queued by the kernel, poll events can be queued while polling events are being processed. As a result, long-running callbacks can allow the poll phase to run much longer than what it was originally intended for.

> Callbacks inside any queue can schedule more callbacks, which makes the queue longer and takes more time than was originally thought.

## Phases Overview

- **pending callbacks**: executes I/O callbacks deferred (postponed or delayed until later) to the next loop iteration.
- **idle, prepare**: only used internally.
- **poll**: retrieve new I/O events; execute I/O related callbacks (almost all, with the exception of close callbacks, the ones scheduled by timers, and `setImmediate()`); Node will block here when appropriate.
- **check**: `setImmediate()` callbacks are invoked here.
- **close callbacks**: some close callbacks, e.g. `socket.on('close', ...)`
- **timers**: this phase executes callbacks scheduled by `setTimeout()` and `setInterval()`.

> **I/O** (Input/Output) refers to any operation that communicates with things outside of your application's CPU and memory. It includes File System, Database queries, Process intercommunication, network requests.

## Phases in Detail

### pending callbacks

This phase executes system-level callbacks that were deferred from a previous loop iteration.

- **Deferred System Operations**: System-level error callbacks that the operating system deferred reporting during the previous iteration.
- **Deferred TCP/UDP Callbacks**: Errors returned by network sockets when attempting connections. For example, if a TCP socket receives an `ECONNREFUSED` error when trying to connect, some Unix systems defer reporting this error until the next tick.
- **Deferred I/O Errors**: Certain low-level disk or network errors that couldn't be executed immediately in the Poll phase due to execution limits or system signals.

#### Example

Imagine a Node.js client attempting to open a socket connection to a server that is offline:

1. The OS attempts the connection during the **Poll** phase.
2. The OS returns an error (`ECONNREFUSED`).
3. Rather than interrupting the current execution flow or executing immediately, libuv queues the callback to be executed at the start of the next loop iteration in the **Pending Callbacks** phase.

### poll

The **poll** phase has two main functions:

1. Calculating how long it should block and poll for I/O, then
2. Processing events in the poll queue.

When the event loop enters the **poll phase**, one of two things will happen:

- If the _poll_ queue _is not empty_, the event loop will iterate through its queue of callbacks, executing them _synchronously_ until either the queue has been exhausted, or the system-dependent hard limit is reached.
- If the _poll_ queue _is empty_, one of the following will happen:
    - If scripts have been scheduled by `setImmediate()`, the event loop will end the **poll** phase and continue to the **check** phase to execute those scheduled scripts.
    - If `setTimeout()` / `setInterval()` timers are ready (whose time thresholds have been reached), the poll phase ends immediately and wraps around to the timers phase.
    - If no timers or `setImmediate` scripts are waiting, Node.js blocks and sleeps in the Poll phase, waiting for incoming callbacks to execute immediately.

> Node.js spends most of its idle time sleeping inside the Poll phase.
> 
> If your server receives no web requests for 10 minutes, the thread remains parked in the Poll phase using OS primitives (like `epoll` on Linux, `kqueue` on macOS, or `IOCP` on Windows) until a packet hits the network interface card or a timer expires.

### check

This phase allows the event loop to execute callbacks immediately after the **poll** phase has completed. If the **poll** phase becomes idle and scripts have been queued with `setImmediate()`, the event loop may continue to the **check** phase rather than waiting.

### close callbacks

This phase is dedicated to running cleanup handlers for resources that were destroyed or closed.

When a stream, socket, or handle is closed abruptly or explicitly via a `.destroy()` or `.close()` call, its `'close'` event callback is emitted during this phase.

These closing callbacks have a special phase to prevent memory leaks and dangling OS resource handles.

### timers

A timer specifies the **threshold** after which a provided callback may be executed. Timer callbacks will run as early as they can be scheduled after the specified amount of time has passed; however, operating system scheduling or the running of other callbacks may delay them.

> Technically, the poll phase controls when timers are executed.

For example, say you schedule a timeout to execute after a 100ms threshold, then your script starts asynchronously reading a file which takes 95ms:

```js
import fs from 'node:fs';

function someAsyncOperation(callback) {
  // Assume this takes 95ms to complete
  fs.readFile('/path/to/file', callback);
}

const timeoutScheduled = Date.now();

setTimeout(() => {
  const delay = Date.now() - timeoutScheduled;
  console.log(`${delay}ms have passed since I was scheduled`);
}, 100);

// do someAsyncOperation which takes 95 ms to complete
someAsyncOperation(() => {
  const startCallback = Date.now();

  // do something that will take 10ms...
  while (Date.now() - startCallback < 10) {
    // do nothing
  }
});
```

When the event loop enters the **poll** phase, it has an empty queue (`fs.readFile()` has not completed), so it will wait for the number of ms remaining until the soonest timer's threshold is reached. While it is waiting, 95ms pass, `fs.readFile()` finishes reading the file, and its callback (which takes 10ms to complete) is added to the **poll** queue and executed. When the callback finishes, there are no more callbacks in the queue, so the event loop will see that the threshold of the soonest timer has been reached, then wrap back to the **timers** phase to execute the timer's callback. In this example, the total delay between the timer being scheduled and its callback being executed will be 105ms.

## `setImmediate()` vs `setTimeout()`

`setImmediate()` and `setTimeout()` are similar, but behave differently depending on when they are called.

- `setImmediate` is designed to execute a script once the current poll phase completes.
- `setTimeout` schedules a script to run after a minimum threshold in ms has elapsed.

The order in which the timers are executed will vary depending on the context in which they are called. If both are called from within the main module, timing will be bound by the performance of the process.

For example, if we run the following script, which is not within an I/O cycle, the order in which the two timers are executed is non-deterministic:

```js
// timeout_vs_immediate.js
setTimeout(() => {
  console.log('timeout');
}, 0);
setImmediate(() => {
  console.log('immediate');
});
```

Output console:

```bash
$ node timeout_vs_immediate.js
timeout
immediate

$ node timeout_vs_immediate.js
immediate
timeout
```

However, if you move the two calls within an I/O cycle, the immediate callback is always executed first:

```js
// timeout_vs_immediate.js
import fs from 'node:fs';
fs.readFile(import.meta.filename, () => {
  setTimeout(() => {
    console.log('timeout');
  }, 0);
  setImmediate(() => {
    console.log('immediate');
  });
});
```

Output console:

```bash
$ node timeout_vs_immediate.js
immediate
timeout

$ node timeout_vs_immediate.js
immediate
timeout
```

The main advantage of using `setImmediate` over `setTimeout` is that `setImmediate` will always be executed before any timers if scheduled within an I/O cycle, independently of how many timers are present.

## process.nextTick

```js
process.nextTick(): void
```

### Understanding `process.nextTick()`

`process.nextTick()` is part of the asynchronous API, but it is not technically part of the event loop. The `nextTickQueue` will be processed after the current operation (synchronous JS code) completes, regardless of the current phase of the event loop.

Looking back at our diagram, any time you call `process.nextTick()` in a given phase, all callbacks passed to `process.nextTick()` will be resolved before the event loop continues. This happens after _every_ callback, not just once per loop iteration, if a nextTick callback itself schedules another `process.nextTick()`, that new callback also drains before the event loop moves on to anything else, including the next callback in the same phase's queue. This is what allows you to "starve" your I/O by making recursive `process.nextTick()` calls, which prevents the event loop from ever reaching the poll phase.

### Why would that be allowed?

Why would something like `process.nextTick()` be allowed in Node.js, given it can starve the event loop? Part of it is a design philosophy where an API should always be asynchronous, even where it doesn't strictly have to be.

```js
function apiCall(arg, callback) {
  if (typeof arg !== 'string') {
    return process.nextTick(
      callback,
      new TypeError('argument should be string')
    );
  }
}
```

The above snippet does an argument check, and if it's not correct, it passes the error to the callback. What we're doing is passing an error back to the user, but only _after_ we have allowed the rest of the user's synchronous code to execute. By using `process.nextTick()` we guarantee that `apiCall` always runs its callback _after_ the rest of the user's code and _before_ the event loop is allowed to proceed.

This philosophy can lead to some potentially problematic situations. Take this snippet, for example:

```js
let bar = null;

// this has an asynchronous signature, but calls callback synchronously
function someAsyncApiCall(callback) {
  callback();
}

// the callback is called before `someAsyncApiCall` completes.
someAsyncApiCall(() => {
  // since someAsyncApiCall hasn't completed, bar hasn't been assigned any value
  console.log('bar', bar); // null
});

bar = 1;
```

`someAsyncApiCall` is supposed to work asynchronously, but it actually operates synchronously. When it is called, the callback provided to `someAsyncApiCall` is called in the same phase of the event loop, which can lead to errors because `bar` has not been initialized yet.

This wouldn't be a problem if `someAsyncApiCall` worked asynchronously (running after the synchronous code has finished executing), which can be achieved by using `process.nextTick()`.

```js
let bar = null;

function someAsyncApiCall(callback) {
  process.nextTick(callback);
}

someAsyncApiCall(() => {
  console.log('bar', bar); // 1
});

bar = 1;
```

## `process.nextTick()` vs `setImmediate()`

We have two calls that are similar as far as users are concerned, but their names are confusing.

- `process.nextTick()` fires immediately, in the same phase.
- `setImmediate()` fires on the following iteration, or "tick," of the event loop.

> Node.js recommends using `setImmediate()` in all cases.

## Why use `process.nextTick()`?

There are two main reasons:

**1. Let users handle errors / cleanup before the event loop moves on**

This ties back to the earlier `apiCall()` example, deferring an error to the callback via `nextTick()` means the user gets a chance to handle it, clean up resources, or retry, all before the event loop proceeds to I/O or timers. It keeps error handling deterministic relative to the rest of your synchronous code.

**2. Run a callback after the call stack unwinds, but before the event loop continues**

Consider the following example:

```js
const server = net.createServer();
server.on('connection', conn => {});

server.listen(8080);
server.on('listening', () => {});
```

If `listen()` binds the port synchronously, and the `'listening'` event were fired via `setImmediate()` instead of `nextTick()`, there'd be a problem: to reach `setImmediate()`'s callback, the event loop has to pass through the poll phase first. That means it's technically possible for an incoming connection to arrive and fire `'connection'` before `'listening'` fires, which is backwards from what a user would expect. Using `nextTick()` guarantees `'listening'` fires immediately after the current operation.