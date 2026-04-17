`useReducer` is a hook in React used for managing complex state logic in a more structured way than `useState`.

Think of `useReducer` like this:

Instead of directly updating state, you:
1. Describe what happened (action)
2. Let a function (reducer) decide how state changes

Syntax:
```js
const [state, dispatch] = useReducer(reducer, initialState);
```
* `state` -> current state
* `dispatch` -> function to send actions
* `reducer` -> function that updates state

A very simple example:
```js
import React, { useReducer } from "react";

// Reducer function
function reducer(state, action) {
  switch (action.type) {
    case "increment":
      return { count: state.count + 1 };
    case "decrement":
      return { count: state.count - 1 };
    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });

  return (
    <>
      <p>{state.count}</p>
      <button onClick={() => dispatch({ type: "increment" })}>
        +
      </button>
      <button onClick={() => dispatch({ type: "decrement" })}>
        -
      </button>
    </>
  );
}
```

Any action must return a state after working on it.

You can combine `useReducer` with **Context API**: [Context API with `useReducer`](Context%20API%20with%20`useReducer`.md)
