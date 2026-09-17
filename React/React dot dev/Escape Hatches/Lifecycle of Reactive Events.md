---
title: Lifecycle of Reactive Events
source: https://react.dev/learn/lifecycle-of-reactive-effects
created: 2026-09-17
tags:
  - react
---
Effects have a different lifecycle from components. Components may mount, update, or unmounts. An effect can only do two things: to start synchronizing something, and later to stop synchronizing it. 

## The lifecycle of an Effect

Every React component goes through the same lifecycle:

- A component _mounts_ when it's added to the screen.
- A component _updates_ when it receives new props or state, usually in response to an interaction.
- A component _unmounts_ when it's removed from the screen. 

Effects are different. An Effect describes how to **synchronize an external system** to the current props and state.

To illustrate this point, consider this Effect connecting your component to a chat server:

```JSX
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
  // ...
}
```

Your Effect's body specifies how to start synchronizing:

```JSX
const connection = createConnection(serverUrl, roomId);
connection.connect();
```

The cleanup function returned by Effect specifies how to stop synchronizing:

```JSX
return () => {
	connection.disconnect();
};
```

One might think that react only synchronizes once when the component mounts and stop synchronizing when the component unmounts. But that is not the case, as there can be multiple cases when your component need to synchronize and stop synchronizing. Say `roomId` in above effect changes due to user's interaction, than our effect would have to re-synchronize. 

## How React re-synchronizes your Effect 

Suppose in our component `roomId` changes from `general` to `travel`. Now our Effect needs to re-synchronize. 

To stop synchronizing, React will call the cleanup function that your Effect returned after connection to the `general` room. Than it will connect or start synchronizing to `travel`. At last when the component unmounts, it will just stop synchronizing, by just calling the cleanup function for the latest `roomId` passed. 

## Thinking from the Effect's perspective

In our chatroom example, our Effect can run multiple times and synchronize with external systems. **We should always focus on a single start/stop cycle at a time. It shouldn’t matter whether a component is mounting, updating, or unmounting. All you need to do is to describe how to start synchronization and how to stop it. If you do it well, your Effect will be resilient to being started and stopped as many times as it’s needed.**

## How React verifies that your Effect can re-synchronize 

By simply unmounting the component once during development. It verifies that you've implemented its cleanup function well.

## How React knows that it needs to re-synchronize the Effect 

The main idea is that it compares the values in the dependency list of your Effect. React will compare values in the dependency array with values that were previously present here (at particular positions) using `Object.is`, and if they differ React will re-synchronize the effect. 

## Effects "react" to reactive values 

The dependency array only needs value which are reactive, the values that are calculated during rendering and participate in the React data flow, like props, state.

If the values don't change like `serverUrl` which probably comes from outside the component, may come from a env file need not be included in the dependency array. 

## What an Effect with empty dependencies means

If the dependency array is empty `[]`, it means that the effect will run or synchronize once when the component mounts and stop synchronizing when the component unmounts. Know that the there would be an extra synchronization cycle in development mode which is done by React to stress test your logic.

## Recap 

- Components can mount, update, and unmount.
- Each Effect has a separate lifecycle from the surrounding component.
- Each Effect describes a separate synchronization process that can _start_ and _stop_.
- When you write and read Effects, think from each individual Effect’s perspective (how to start and stop synchronization) rather than from the component’s perspective (how it mounts, updates, or unmounts).
- Values declared inside the component body are “reactive”.
- Reactive values should re-synchronize the Effect because they can change over time.
- The linter verifies that all reactive values used inside the Effect are specified as dependencies.
- All errors flagged by the linter are legitimate. There’s always a way to fix the code to not break the rules.