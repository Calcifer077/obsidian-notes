---
title: You might not need an Effect
source: https://react.dev/learn/you-might-not-need-an-effect
created: 2026-09-13
tags:
  - react
---
Effects are an escape hatch from React paradigm. They let you "step outside" of React and synchronize your components with some external system like a non-React widget, network or the browser DOM. 

## How to remove unnecessary Effects 

There are two common cases in which you don't need Effects:

- You don't need Effects to transform data for rendering.
- You don't need Effects to handle user events.

You do need Effects to synchronize with external systems.

## Caching expensive calculations 

If you have to do some calculations of state, say like filtering of items. Instead of doing it in a effect (is possible, as it will run after any state in the dependency array changes), you can use `filter` to do so. And if the calculation is expensive you can cache or memoize by wrapping it in a `useMemo` hook:

```JSX
import { useMemo, useState } from 'react';

function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  const visibleTodos = useMemo(() => {
    // ✅ Does not re-run unless todos or filter change
    return getFilteredTodos(todos, filter);
  }, [todos, filter]);
  // ...
}
```

This tells React that you don't want the inner function to re-run unless either `todos` or `filter` have changed. React will remember the return value of `getFilteredTodos()` during the initial render. During the next renders, it will check if `todos` or `filter` are different. If they’re the same as last time, `useMemo` will return the last result it has stored. But if they are different, React will call the inner function again (and store its result).

`useMemo` runs during rendering, so this only works for pure calculations. 

## Sending a POST request 

The `Form` component below sends two kinds of POST requests. It sends an analytics event when it mounts. When you fill in the form and click the Submit button, it will send a POST request to the `/api/register` endpoint:

```JSX
function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  // ✅ Good: This logic should run because the component was displayed
  useEffect(() => {
    post('/analytics/event', { eventName: 'visit_form' });
  }, []);

  // 🔴 Avoid: Event-specific logic inside an Effect
  const [jsonToSubmit, setJsonToSubmit] = useState(null);
  useEffect(() => {
    if (jsonToSubmit !== null) {
      post('/api/register', jsonToSubmit);
    }
  }, [jsonToSubmit]);

  function handleSubmit(e) {
    e.preventDefault();
    setJsonToSubmit({ firstName, lastName });
  }
  // ...
}
```

The first POST request is valid because it is being fired in response to the component mounting on the screen. The second request is in respect to a button click, so it can be shifted to a event handler instead.