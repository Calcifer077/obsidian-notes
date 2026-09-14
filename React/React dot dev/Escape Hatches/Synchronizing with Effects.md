---
title: Synchronizing with Effects
source: https://react.dev/learn/synchronizing-with-effects
created: 2026-09-12
tags:
  - react
---
Some components need to synchronize with external systems. _Effects_ let you run some code after rendering so that you can synchronize your component with some system outside of React.

## What are Effects and how are they different from events?

Before getting to Effects, you need to be familiar with two types of logic inside React components:

- **Rendering code**, lives at top level of your component. This is only concerned about what will be displayed on the screen using JSX. This must be pure.
- **Event handlers**, are nested functions inside your components that do things (like HTTP post, or navigate the user) on some event like a click rather than just calculate them.

Sometimes this isn't enough. Consider a `ChatRoom` component that must connect to the chat server whenever it's visible on the screen. Connecting to a server isn't a pure calculation and it won't happen after a particular event like a click. 

**_Effects_ let you specify side effects that are caused by rendering itself, rather than by a particular event.** Effects run at the end of a commit after the screen updates. This is a good time to synchronize the react components with some external system. 

## How to write an Effect 

To write an Effect, follow these three steps:

1. Declare an Effect. By default, your Effect will run after every commit. 
2. Specify the Effect dependencies.
3. Add cleanup if needed. 

### Step 1: Declare an Effect

To declare an Effect in your component, import the `useEffect` hook from react:

```JSX
import { useEffect } from 'react';
```

Then, call it at the top level of your component:

```JSX
function MyComponent() {
  useEffect(() => {
    // Code here will run after *every* render
  });
  return <div />;
}
```

Every time your component renders, React will update the screen and then run the code inside `useEffect`. In other words, **`useEffect` "delays" a piece of code from running until that render is reflected on the screen.** 

### Step 2: Specify the Effect dependencies 

By default, Effects run after every render, Often, this is not what you want.

You can tell React to skip unnecessarily re-running the Effect by specifying an array of dependencies as the second argument to the `useEffect` call. This dependency array can be added as follows:

```JSX
useEffect(() => {
	// ...
}, []);
```

If the code inside of your Effect depends on some prop, add that prop to the dependency array. 

```JSX
  useEffect(() => {
    if (isPlaying) { // It's used here...
      // ...
    } else {
      // ...
    }
  }, [isPlaying]); // ...so it must be declared here!
```

Here, `isPlaying` is a prop. Specifying `[isPlaying]` as the dependency array tells React that it should skip re-running your Effect if `isPlaying` is the same as it was during the previous render.

The dependency array can contain multiple dependencies. React will only skip re-running the Effect if _all_ the dependencies you specify have exactly the same values as they had during the previous render. React compares the dependency values using `Object.is` comparison. 

>You can skip refs from dependency array even if you were using it in code inside effect. It is because react guarantees that you'll always get the same object form the same `useRef` call on every render. It never changes. 

### Step 3: Add cleanup if needed 

You can add cleanup code (code that needs to be run when your component unmounts (is no longer visible on the screen)) as follows:

```JSX
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []);
```

When your component mounts, is visible for the first time, above effect will run, and when the component unmounts, the cleanup will run.  

React will call your cleanup function each time before the Effect runs again, and one final time when the component unmounts (gets removed).

> In development mode, effects run twice due to strict mode. It won't happen in production. 

## How to handle the Effect firing twice in development?

**The right question isn’t “how to run an Effect once”, but “how to fix my Effect so that it works after remounting”.**

Usually, the answer is to implement the cleanup function. The cleanup function should stop or undo whatever the Effect was doing. The rule of thumb is that the user shouldn’t be able to distinguish between the Effect running once (as in production) and a _setup → cleanup → setup_ sequence (as you’d see in development).

Most of the Effects you'll write will fit into one of the common patterns below.

### Controlling non-React widgets 

Sometimes you need to add UI widgets that aren’t written in React. For example, let’s say you’re adding a map component to your page. It has a `setZoomLevel()` method, and you’d like to keep the zoom level in sync with a `zoomLevel` state variable in your React code. Your Effect would look similar to this:

```JSX
useEffect(() => {
  const map = mapRef.current;
  map.setZoomLevel(zoomLevel);
}, [zoomLevel]);
```

Note that there is no cleanup needed in this case because it wouldn't hurt to set the same zoom level twice.

Some APIs may not allow you to call them twice in a row. For example, the [`showModal`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/showModal) method of the built-in [`<dialog>`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement) element throws if you call it twice. Implement the cleanup function and make it close the dialog:

```JSX
useEffect(() => {
  const dialog = dialogRef.current;
  dialog.showModal();
  return () => dialog.close();
}, []);
```

### Subscribing to events 

If your effect subscribes to something, the cleanup function should unsubscribe:

```JSX
useEffect(() => {
  function handleScroll(e) {
    console.log(window.scrollX, window.scrollY);
  }
  window.addEventListener('scroll', handleScroll);
  return () => window.removeEventListener('scroll', handleScroll);
}, []);
```

### Triggering animations 

If your Effect animates something in, the cleanup function should reset the animation to the initial values:

```JSX
useEffect(() => {
  const node = ref.current;
  node.style.opacity = 1; // Trigger the animation
  return () => {
    node.style.opacity = 0; // Reset to the initial value
  };
}, []);
```

### Fetching data 

If your Effect fetches something, the cleanup function should either [abort the fetch](https://developer.mozilla.org/en-US/docs/Web/API/AbortController) or ignore its result:

```JSX
useEffect(() => {
  let ignore = false;

  async function startFetching() {
    const json = await fetchTodos(userId);
    if (!ignore) {
      setTodos(json);
    }
  }

  startFetching();

  return () => {
    ignore = true;
  };
}, [userId]);
```

You can’t “undo” a network request that already happened, but your cleanup function should ensure that the fetch that’s _not relevant anymore_ does not keep affecting your application. If the `userId` changes from `'Alice'` to `'Bob'`, cleanup ensures that the `'Alice'` response is ignored even if it arrives after `'Bob'`.

### Not an Effect: Initializing the application 

Some logic should only run once when the application starts. You can put it outside your components:

```JSX
if (typeof window !== 'undefined') { // Check if we're running in the browser.
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
```

This guarantees that such logic only runs once after the browser loads the page. 

## Recap 

- Unlike events, Effects are caused by rendering itself rather than a particular interaction.
- Effects let you synchronize a component with some external system (third-party API, network, etc).
- By default, Effects run after every render (including the initial one).
- React will skip the Effect if all of its dependencies have the same values as during the last render.
- You can’t “choose” your dependencies. They are determined by the code inside the Effect.
- Empty dependency array (`[]`) corresponds to the component “mounting”, i.e. being added to the screen.
- In Strict Mode, React mounts components twice (in development only!) to stress-test your Effects.
- If your Effect breaks because of remounting, you need to implement a cleanup function.
- React will call your cleanup function before the Effect runs next time, and during the unmount.