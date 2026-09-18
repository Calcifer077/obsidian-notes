---
title:
  - Reusing Logic with Custom Hooks
source: https://react.dev/learn/reusing-logic-with-custom-hooks
created: 2026-09-18
tags:
  - react
---
## Custom Hooks: Sharing logic between components 

Let's consider a simple hook that tracks whether the user is online or not. It would need two things:

1. A piece of state that tracks whether the network is online.
2. An Effect that subscribes to the global `online` and `offline` events, and updates that state.

At the very basic a component having this functionality would look like this:

```JSX
import { useState, useEffect } from 'react';

export default function StatusBar() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
}
```

If other component's need to use this functionality, we can extract this into a custom hook.

## Extracting your own custom Hook from a component 

Imagine for a moment that, similar to `useState` and `useEffect`, there was a built-in `useOnlineStatus` Hook. Then we could have done something like this:

```JSX
const isOnline = useOnlineStatus();
```

To write such a hook, we can declare a function called `useOnlineStatus` and move all the related code into it:

```JSX
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  return isOnline;
}
```

At the end of the function, we return `isOnline`, which lets other component read this value.

## Hook names always start with `use`

You must follow following naming conventions for components and hooks:

1. **React component names must start with a capital letter**, like `StatusBar` and `SaveButton`. React components also need to return something that React knows how to display, like a piece of JSX.
2. **Hook names must start with `use` followed by a capital letter**, like `useState` (built-in) or `useOnlineStatus` (custom).

## Custom Hooks let you share stateful logic, not state itself 

If we used the above custom hook like below:

```JSX
function StatusBar() {
  const isOnline = useOnlineStatus();
  // ...
}

function SaveButton() {
  const isOnline = useOnlineStatus();
  // ...
}
```

Both these components get a separate value of `isOnline` and doesn't depend on each other. A change in one don't reflect in other.

## Passing reactive values between Hooks 

The code inside your custom Hooks will re-run during every re-render of your component. This is why, like components, custom Hooks need to be pure.

Because custom Hooks re-render together with your component, they always receive the latest props and state. Changes to passed state and props to hooks will also cause Effect to re-run meaning that every time your component re-renders, the hook will also re-run. 

## Passing event handlers to custom Hooks 

Suppose you want to define custom logic in your effect inside your custom hook based on the component that calls it. You can simply pass this logic (event handler) to your custom hook which can be further used by your effect.

There is one minor improvement you can make here, as you are using the event handler inside your effect, you would have to mention it in effect's dependency array. To remove this from dependency array you can use a Effect Event.

To look at code example: [Check docs](https://react.dev/learn/reusing-logic-with-custom-hooks#passing-event-handlers-to-custom-hooks)

## Recap 

- Custom Hooks let you share logic between components.
- Custom Hooks must be named starting with `use` followed by a capital letter.
- Custom Hooks only share stateful logic, not state itself.
- You can pass reactive values from one Hook to another, and they stay up-to-date.
- All Hooks re-run every time your component re-renders.
- The code of your custom Hooks should be pure, like your component’s code.
- Wrap event handlers received by custom Hooks into Effect Events.
- Don’t create custom Hooks like `useMount`. Keep their purpose specific.
- It’s up to you how and where to choose the boundaries of your code.