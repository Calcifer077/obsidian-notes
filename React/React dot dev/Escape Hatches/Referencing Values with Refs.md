---
title: Referencing Values with Refs
source: https://react.dev/learn/referencing-values-with-refs
created: 2026-09-12
tags:
  - react
---
When you want a component to "remember" some information, but you don't want that information to trigger new renders, you can use a `ref`.

## Adding a ref to your component

You can add a ref to your component by importing `useRef` hook from react:

```JSX
import { useRef } from 'react';
```

Inside your component, you can just call `useRef` hook and pass the initial value:

```JSX
const res = useRef(0);
```

`useRef` returns an object like this:

```JSX
{
	current: 0 // The value you passed to useRef
}
```

`ref.current` can hold anything, ranging from strings to functions. 

This value is mutable, meaning you can both read and write to it. Modifications made to this value won't trigger a re-render.

## Example: building a stopwatch

To build a stopwatch, we can simply use `setInterval` which updates time (state) after some set ms. But to stop this stopwatch we need access to the interval ID. To store this intervalID we can use `ref` as this ID needs to remain consistent between re-renders.

```JSX
import { useState, useRef } from 'react';

export default function Stopwatch() {
  const [startTime, setStartTime] = useState(null);
  const [now, setNow] = useState(null);
  const intervalRef = useRef(null);

  function handleStart() {
    setStartTime(Date.now());
    setNow(Date.now());

    clearInterval(intervalRef.current);
    intervalRef.current = setInterval(() => {
      setNow(Date.now());
    }, 10);
  }

  function handleStop() {
    clearInterval(intervalRef.current);
  }

  let secondsPassed = 0;
  if (startTime != null && now != null) {
    secondsPassed = (now - startTime) / 1000;
  }

  return (
    <>
      <h1>Time passed: {secondsPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>
        Start
      </button>
      <button onClick={handleStop}>
        Stop
      </button>
    </>
  );
}
```

## Differences between refs and state 


| **refs**                                                                              | **state**                                                                                                                                                |
| ------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `useRef(initialValue)` returns `{ current: initialValue }`                            | `useState(initialValue)` returns the current value of a state variable and a state setter function ( `[value, setValue]`)                                |
| Doesn’t trigger re-render when you change it.                                         | Triggers re-render when you change it.                                                                                                                   |
| Mutable—you can modify and update `current`’s value outside of the rendering process. | ”Immutable”—you must use the state setting function to modify state variables to queue a re-render.                                                      |
| You shouldn’t read (or write) the `current` value during rendering.                   | You can read state at any time. However, each render has its own [snapshot](https://react.dev/learn/state-as-a-snapshot) of state which does not change. |
### How does `useRef` work inside?

In principle, `useRef` could be implemented on top of `useState`. It just doesn't return the state setter function.

```JSX
// Inside of React
function useRef(initialValue) {
  const [ref, unused] = useState({ current: initialValue });
  return ref;
}
```

## When to use refs 

Typically you will use a ref when your components need to communicate with external APIs - often a browser API that won't impact the appearance of the component.

- Storing timeout IDs 
- Storing and manipulating DOM elements 
- Storing other objects that aren't necessary to calculate the JSX 

If your component needs to store some value, but it doesn't impact the rendering logic, choose refs. 

## Best practices for refs 

- **Treat refs as an escape hatch**. Try to use refs only when you work with external systems or browser APIs.
- **Don't read or write `ref.current` during rendering**. Only exception being code like `if (!ref.current) ref.current = new Thing()` which only sets the ref once during the first render.

Limitations of React state don’t apply to refs. For example, state acts like a [snapshot for every render](https://react.dev/learn/state-as-a-snapshot) and [doesn’t update synchronously.](https://react.dev/learn/queueing-a-series-of-state-updates) But when you mutate the current value of a ref, it changes immediately:

```
ref.current = 5;console.log(ref.current); // 5
```

This is because **the ref itself is a regular JavaScript object,** and so it behaves like one.

You can also do mutation on refs as long as the object being mutated is not used for rendering.

## Refs and the DOM 

The most common use case for a ref is to access a DOM element. 

## Recap

- Refs are an escape hatch to hold onto values that aren’t used for rendering. You won’t need them often.
- A ref is a plain JavaScript object with a single property called `current`, which you can read or set.
- You can ask React to give you a ref by calling the `useRef` Hook.
- Like state, refs let you retain information between re-renders of a component.
- Unlike state, setting the ref’s `current` value does not trigger a re-render.
- Don’t read or write `ref.current` during rendering. This makes your component hard to predict.