---
title: State as a Snapshot
source: https://react.dev/learn/state-as-a-snapshot
created: 2026-08-02
tags:
  - react
---
## Setting state triggers renders

Whenever you change (set) state in react, it triggers a render.

```JSX
import { useState } from 'react';

export default function Form() {
  const [isSent, setIsSent] = useState(false);
  const [message, setMessage] = useState('Hi!');
  if (isSent) {
    return <h1>Your message is on its way!</h1>
  }
  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      setIsSent(true);
      sendMessage(message);
    }}>
      <textarea
        placeholder="Message"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button type="submit">Send</button>
    </form>
  );
}

function sendMessage(message) {
  // ...
}
```

Here's what happens when you click the button:
1. The `onSubmit` event handler executes.
2. `setIsSent(true)` sets `isSent` to `true` and queues a new render.
3. React re-renders the component according to the new `isSent` value.

## Rendering takes a snapshot in time

[“Rendering”](https://react.dev/learn/render-and-commit#step-2-react-renders-your-components) means that React is calling your component, which is a function. The JSX you return from that function is like a snapshot of the UI in time. Its props, event handlers, and local variables were all calculated **using its state at the time of the render**.

When React re-renders a component:

1. React calls your function again.
2. Your function returns a new JSX snapshot.
3. React then updates the screen to match the snapshot your function returned.

![](../../../assets/Pasted%20image%2020260802212659.png)

As a component’s memory, state is not like a regular variable that disappears after your function returns. State actually “lives” in React itself—as if on a shelf!—outside of your function. When React calls your component, it gives you a snapshot of the state for that particular render. Your component returns a snapshot of the UI with a fresh set of props and event handlers in its JSX, all calculated **using the state values from that render!**

![](../../../assets/Pasted%20image%2020260802213048.png)

Here's a little experiment to show you how this works. In below example, one might expect that clicking "+3" button would increment the counter three times because it calls `setNumber(number + 1)` three times.

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

Notice that `number` only increments once per click!

**Setting state only changes it for the next render**. During the first render, `number` was `0`. This is why, in that render's `onClick` handler, the value of `number` is still `0` even after `setNumber(number + 1)` was called.

Here is what this  button's click handler tells React to do:

1. `setNumber(number + 1)`: `number` is `0` so `setNumber(0 + 1)`.
    - React prepares to change `number` to `1` on the next render.
2. `setNumber(number + 1)`: `number` is `0` so `setNumber(0 + 1)`.
    - React prepares to change `number` to `1` on the next render.
3. `setNumber(number + 1)`: `number` is `0` so `setNumber(0 + 1)`.
    - React prepares to change `number` to `1` on the next render.

Even though you called `setNumber(number + 1)` three times, in _this render’s_ event handler `number` is always `0`, so you set the state to `1` three times. This is why, after your event handler finishes, React re-renders the component with `number` equal to `1` rather than `3`.

React will only render again after the final state update all the earlier state update will be ignored. 

A more concrete example would be:

```JSX
onClick={() => {
	setNumber(number + 1);
	setNumber(number + 2);
	setNumber(number + 3);
}}
```

After all the state setters finishes `+3` will be the only one that prevails. 

## State over time

What if we put the state in a async function, will it show updated state or the past state.

```JSX
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        setTimeout(() => {
          alert(number);
        }, 3000);
      }}>+5</button>
    </>
  )
}
```

The `alert` will run after react has finished rendering with updated state say `5`. But the `alert` will still show `0`. The state stored in React may have changed by the time alert runs, but it was scheduled using a snapshot of the state at the time the user interacted with it (still 0).

We can use the substitution method to check.

```JSX
setNumber(0 + 5);
setTimeout(() => {
  alert(0);
}, 3000);
```

**A state variable’s value never changes within a render,** even if its event handler’s code is asynchronous. Inside _that render’s_ `onClick`, the value of `number` continues to be `0` even after `setNumber(number + 5)` was called. Its value was “fixed” when React “took the snapshot” of the UI by calling your component.

## Recap 

- Setting state requests a new render.
- React stores state outside of your component, as if on a shelf.
- When you call `useState`, React gives you a snapshot of the state _for that render_.
- Variables and event handlers don’t “survive” re-renders. Every render has its own event handlers.
- Every render (and functions inside it) will always “see” the snapshot of the state that React gave to _that_ render.
- You can mentally substitute state in event handlers, similarly to how you think about the rendered JSX.
- Event handlers created in the past have the state values from the render in which they were created.