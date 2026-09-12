---
title: Manipulating the DOM with Refs
source: https://react.dev/learn/manipulating-the-dom-with-refs
created: 2026-09-12
tags:
  - react
---
React automatically updates the DOM to match your render output, so your components won't often need to manipulate it. However, sometimes you might need access to the DOM elements managed by React - for example, to focus a node, scroll to it. There is no built-in way to do those things in react, so you will need a _ref_ to the DOM node. 

## Getting a ref to the Node 

To access a DOM node managed by React, first, import the `useRef` hook:

```JSX
import { useRef } from 'react';
```

Then, use it to declare a ref inside your component:

```JSX
const myRef = useRef(null);
```

Finally, pass your ref as the `ref` attribute to the JSX tag for which you want to get the DOM node:

```JSX
<div ref={myRef}>
```

The `useRef` hook returns an object with a single property called `current`. Initially, `myRef.current` will be `null`. When react creates a DOM node for this `<div>`, react will put a reference to this node into `myRef.current`. You can then access this DOM node from your event handlers and use the built-in browser APIs defined on it.

## Example: Focusing a text input 

In the below example, clicking the button will focus the input:

```JSX
import { useRef } from 'react';

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```

Initially, we declare a ref called `inputRef` and point it to a DOM element. Now we can call any browser API on it, so we just call `inputRef.current.focus()`.

### How to manage a list of refs using a ref callback 

Suppose you have multiple DOM elements or some other use case which you don't know the exact number for and wish to attach a ref to each of them. Something like this wouldn't work:

```JSX
<ul>
  {items.map((item) => {
    // Doesn't work!
    const ref = useRef(null);
    return <li ref={ref} />;
  })}
</ul>
```

This is because **hooks must only be called at the top-level of your component**.

One possible way around this is to get a single ref to their parent element, and then use DOM manipulation methods like `querySelectorAll` to "find" the individual child nodes from it.

Another solution is to pass a function to the `ref` attribute. This is called a ref callback. React will call your ref callback with the DOM node when it's time to set the ref, and call the cleanup function returned from the callback when it's time to clear it. This lets you maintain your own array of a Map, and access any ref by its index or some kind of ID.

Below is a example, to use scroll to an arbitrary node in a long list:

```JSX
import { useRef, useState } from "react";

export default function CatFriends() {
  const itemsRef = useRef(null);
  const [catList, setCatList] = useState(setupCatList);

  function scrollToCat(cat) {
    const map = getMap();
    const node = map.get(cat);
    node.scrollIntoView({
      behavior: "smooth",
      block: "nearest",
      inline: "center",
    });
  }

  function getMap() {
    if (!itemsRef.current) {
      // Initialize the Map on first usage.
      itemsRef.current = new Map();
    }
    return itemsRef.current;
  }

  return (
    <>
      <nav>
        <button onClick={() => scrollToCat(catList[0])}>Neo</button>
        <button onClick={() => scrollToCat(catList[5])}>Millie</button>
        <button onClick={() => scrollToCat(catList[8])}>Bella</button>
      </nav>
      <div>
        <ul>
          {catList.map((cat) => (
            <li
              key={cat.id}
              ref={(node) => {
                const map = getMap();
                map.set(cat, node);

                return () => {
                  map.delete(cat);
                };
              }}
            >
              <img src={cat.imageUrl} />
            </li>
          ))}
        </ul>
      </div>
    </>
  );
}

function setupCatList() {
  const catCount = 10;
  const catList = new Array(catCount)
  for (let i = 0; i < catCount; i++) {
    let imageUrl = '';
    if (i < 5) {
      imageUrl = "https://placecats.com/neo/320/240";
    } else if (i < 8) {
      imageUrl = "https://placecats.com/millie/320/240";
    } else {
      imageUrl = "https://placecats.com/bella/320/240";
    }
    catList[i] = {
      id: i,
      imageUrl,
    };
  }
  return catList;
}
```

In the above example, `itemsRef` doesn't hold a single DOM node. Instead it holds a Map. The `ref` callback on every list item takes care to update the map and also handles the cleanup. 

## Accessing another component's DOM nodes 

You can pass refs from parent component to child components just like any other props. 

```JSX
import { useRef } from 'react';

function MyInput({ ref }) {
  return <input ref={ref} />;
}

function MyForm() {
  const inputRef = useRef(null);
  return <MyInput ref={inputRef} />
}
```

You can control functionality of `input` in `MyInput` from `MyForm` using say a click handler:

```JSX
import { useRef } from 'react';

function MyInput({ ref }) {
  return <input ref={ref} />;
}

export default function MyForm() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```

### Only exposing a subset of API 

In the above example, the parent component can do anything with the ref like change its CSS styles. In some cases, you may want to restrict the exposed functionality. You can do that with `useImperativeHandle`:

```JSX
import { useRef, useImperativeHandle } from "react";

function MyInput({ ref }) {
  const realInputRef = useRef(null);
  useImperativeHandle(ref, () => ({
    // Only expose focus and nothing else
    focus() {
      realInputRef.current.focus();
    },
  }));
  return <input ref={realInputRef} />;
};

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>Focus the input</button>
    </>
  );
}
```

## When React attaches the refs 

In react, every update is split in two phases:

- During render, React calls your components to figure out what should be on the screen.
- During commit, React applies changes to the DOM. 

In general, you don’t want to access refs during rendering. That goes for refs holding DOM nodes as well. During the first render, the DOM nodes have not yet been created, so `ref.current` will be `null`. And during the rendering of updates, the DOM nodes haven’t been updated yet. So it’s too early to read them.

React sets `ref.current` during the commit. Before updating the DOM, React sets the affected `ref.current` values to `null`. After updating the DOM, React immediately sets them to the corresponding DOM nodes.

### Flushing state updates synchronously with `flushSync`

In React, **state updates are queued/batched**, meaning React may wait until the current event handler finishes before processing the updates and re-rendering the component. 

However, you may not want that. You may want the updated state synchronously. To do this, import `flushSync` from `react-dom` and wrap the state update into a `flushSync` call:

```JSX
flushSync(() => {
  setTodos([ ...todos, newTodo]);
});
listRef.current.lastChild.scrollIntoView();
```

If `listRef` was pointing to `todos`, it would see the updated state without waiting for a re-render. 

## Best practices for DOM manipulation with refs 

Refs are an escape hatch. You should (try to) only use them when you have to "step outside React". Common examples include managing focus, scroll position, or calling browser APIs that React does not expose. 

## Recap

- Refs are a generic concept, but most often you’ll use them to hold DOM elements.
- You instruct React to put a DOM node into `myRef.current` by passing `<div ref={myRef}>`.
- Usually, you will use refs for non-destructive actions like focusing, scrolling, or measuring DOM elements.
- A component doesn’t expose its DOM nodes by default. You can opt into exposing a DOM node by using the `ref` prop.
- Avoid changing DOM nodes managed by React.
- If you do modify DOM nodes managed by React, modify parts that React has no reason to update.
