---
title: Queueing a Series of State Updates
source: https://react.dev/learn/queueing-a-series-of-state-updates
created: 2026-08-03
tags:
  - react
---
## React batches state updates

In the below example, one might expect that clicking the "+3" button will increment the counter three times because it calls `setNumber(number + 1)` three times:

```JSX
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 1);
        setNumber(number + 1);
        setNumber(number + 1);
      }}>+3</button>
    </>
  )
}
```

However, as you might recall from the previous note [each render's state values are fixed](State%20as%20a%20Snapshot.md), so that value of `number` inside the first render's event handler is always `0`, no matter how many times you call `setNumber(1)`.

```JSX
setNumber(0 + 1);
setNumber(0 + 1);
setNumber(0 + 1);
```

But there is one other factor at play here. **React waits until _all_ code in the event handlers has run before processing your state updates.** This is why the re-render only happens _after_ all these `setNumber()` calls.

This might remind you of a waiter taking an order at the restaurant. A waiter doesn’t run to the kitchen at the mention of your first dish! Instead, they let you finish your order, let you make changes to it, and even take orders from other people at the table.

This lets you update multiple state variables - even from multiple components - without triggering too many re-renders. But this also means that the UI won't be updated until after your event handler, and any code in it, completes. This behaviour, also know as **batching**, makes your React app run much faster.

**React does not batch across multiple intentional events like clicks** - each click is handled separately.

## Updating the same state multiple times before the next render

It is an uncommon use case, but if you would like to update the same state variable multiple times before the next render, instead of passing the _next state value_ like `setNumber(number + 1)`, you can pass a _function_ that calculates the next state based on the previous one in the queue, like `setNumber(n => n + 1)`. It is a way to tell React to “do something with the state value” instead of just replacing it.

```JSX
<button onClick={() => {
	setNumber(n => n + 1);
	setNumber(n => n + 1);
	setNumber(n => n + 1);
}}>+3</button>
```

Here, `n => n + 1` is called an **updater function**. When you pass it to a state setter:

1. React queues this function to be processed after all the other code in the event handler has run.
2. During the next render, React goes through the queue and gives you the final updated state.

```JSX
setNumber(n => n + 1);  
setNumber(n => n + 1);  
setNumber(n => n + 1);
```

Here’s how React works through these lines of code while executing the event handler:

1. `setNumber(n => n + 1)`: `n => n + 1` is a function. React adds it to a queue.
2. `setNumber(n => n + 1)`: `n => n + 1` is a function. React adds it to a queue.
3. `setNumber(n => n + 1)`: `n => n + 1` is a function. React adds it to a queue.

When you call `useState` during the next render, React goes through the queue. The previous `number` state was `0`, so that's what React passes to the first updater function as the `n` argument. Then React takes the return value of your previous updater function and passes it to the next updater as `n`, and so on:

|queued update|`n`|returns|
|---|---|---|
|`n => n + 1`|`0`|`0 + 1 = 1`|
|`n => n + 1`|`1`|`1 + 1 = 2`|
|`n => n + 1`|`2`|`2 + 1 = 3`|

React store `3` as the final result and returns it from `useState`.

This why clicking "+3" in the above example correctly increments the value by 3. 

_Note: the main point behind updater function is that it tells react to not just simply replace the value with something else, but update it according to the previous state. So if you have multiple updater function, the state of the previous updater function is passed to the current updater function and so on, until the entire queue is processed and ultimately return the result to your next render's `useState`._

## What happens if you update state after replacing it

What about this event handler? 

```JSX
<button onClick={() => {
  setNumber(number + 5);
  setNumber(n => n + 1);
}}>
```

Here, the first is a setter function which simply change the state to `number + 5`, and than we have a updater function which uses the updated value.

Here’s what this event handler tells React to do:

1. `setNumber(number + 5)`: `number` is `0`, so `setNumber(0 + 5)`. React adds _“replace with `5`”_ to its queue.
2. `setNumber(n => n + 1)`: `n => n + 1` is an updater function. React adds _that function_ to its queue.

During the next render, React goes through the state queue:

|queued update|`n`|returns|
|---|---|---|
|”replace with `5`”|`0` (unused)|`5`|
|`n => n + 1`|`5`|`5 + 1 = 6`|

React stores `6` as the final result and returns it from `useState`.

_Note: We can say that `setState(5)` is like `setState(n => 5)`, but `n` is unused._

## What happens if you replace state after updating it

What happens in this case?

```JSX
<button onClick={() => {
  setNumber(number + 5);
  setNumber(n => n + 1);
  setNumber(42);
}}>
```

Here’s how React works through these lines of code while executing this event handler:

1. `setNumber(number + 5)`: `number` is `0`, so `setNumber(0 + 5)`. React adds _“replace with `5`”_ to its queue.
2. `setNumber(n => n + 1)`: `n => n + 1` is an updater function. React adds _that function_ to its queue.
3. `setNumber(42)`: React adds _“replace with `42`”_ to its queue.

During the next render, React goes through the state queue:

|queued update|`n`|returns|
|---|---|---|
|”replace with `5`”|`0` (unused)|`5`|
|`n => n + 1`|`5`|`5 + 1 = 6`|
|”replace with `42`”|`6` (unused)|`42`|

Then React stores `42` as the final result and returns it from `useState`.

To summarize, here's how you can think of what you're passing to the `setNumber` state setter:

- **An updater function** (e.g. `n => n + 1`) gets added to the queue.
- **Any other value** (e.g. number `5`) adds "replace with `5`" to the queue, ignoring what's already queued.

After the event handler completes, React will trigger a re-render. During the re-render, React will process the queue. Updater functions run during rendering, so **updater functions must be [pure](https://react.dev/learn/keeping-components-pure)** and only _return_ the result. Don’t try to set state from inside of them or run other side effects. In Strict Mode, React will run each updater function twice (but discard the second result) to help you find mistakes.

## Recap 

- Setting state does not change the variable in the existing render, but it requests a new render.
- React processes state updates after event handlers have finished running. This is called batching.
- To update some state multiple times in one event, you can use `setNumber(n => n + 1)` updater function.