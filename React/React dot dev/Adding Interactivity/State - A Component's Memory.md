---
title: "State: A Component's Memory"
source: https://react.dev/learn/state-a-components-memory
created: 2026-08-02
tags:
  - react
---
State → component specific memory

## When a regular variable isn't enough

Here's a component that renders a sculpture image. Clicking the "Next" button should show the next sculpture by changing `index` to `1`, then `2`, and so on. However this won't work.

```JSX
import { sculptureList } from './data.js';

export default function Gallery() {
  let index = 0;

  function handleClick() {
    index = index + 1;
  }

  let sculpture = sculptureList[index];
  return (
    <>
      <button onClick={handleClick}>
        Next
      </button>
      <h2>
        <i>{sculpture.name} </i>
        by {sculpture.artist}
      </h2>
      <h3>
        ({index + 1} of {sculptureList.length})
      </h3>
      <img
        src={sculpture.url}
        alt={sculpture.alt}
      />
      <p>
        {sculpture.description}
      </p>
    </>
  );
}
```

The `handleClick` event handler is updating a local variable, `index`. But two things prevent that change from being visible:

1. **Local variables don't persist between renders**
2. **Changes to local variables won't trigger renders**

To update a component with new data, two things need to happen:

1. **Retain** the data between renders.
2. **Trigger** React to render the component with new data (re-rendering).

The [`useState`](https://react.dev/reference/react/useState) Hook provides those two things:

1. A **state variable** to retain the data between renders.
2. A **state setter function** to update the variable and trigger React to render the component again.

## Adding a state variable

To add a state variable, import `useState` from React at the top of the file:

```jsx
import { useState } from 'react';
```

Then, replace this line:

```JSX
let index = 0;
```

with

```JSX
const [index, setIndex] = useState(0);
```

`index` is a state variable and `setIndex` is the setter function.

This is how they work together in `handleClick`:

```JSX
function handleClick(){
	setIndex(index + 1);
}
```

## Meet your first hook

In React, `useState`, as well as any other function starting with "`use`", is called a hook.

_Hooks_ are special functions that are only available while React is rendering. They let you "hook into" different React features.

_Note: **Hooks** - functions starting with `use` - **can only be called at the top level** of your components or in your own hooks.. You can't call Hooks inside conditions, loops, or other nested function._

## Anatomy of `useState`

When you call `useState`, you are telling React that you want this component to remember something:

```JSX
const [index, setIndex] = useState(0);
```

In this case, you want React to remember `index`.

_Note: The convention is to name this pair like `const [something, setSomething]`._

The only argument to `useState` is the initial value of your state variable. In this example, the `index`'s initial value is set to `0` with `useState(0).`

Here' how that happens in action:

```JSX
const [index, setIndex] = useState(0);
```

1. **Your component renders the first time.** Because you passed `0` to `useState` as the initial value for `index`, it will return `[0, setIndex]`. React remembers `0` is the latest state value.
2. **You update the state.** When a user clicks the button, it calls `setIndex(index + 1)`. `index` is `0`, so it’s `setIndex(1)`. This tells React to remember `index` is `1` now and triggers another render.
3. **Your component’s second render.** React still sees `useState(0)`, but because React _remembers_ that you set `index` to `1`, it returns `[1, setIndex]` instead.
4. And so on!

You can as many state as you want for a component.

### How does React know which state to return?

You might have noticed that the `useState` call does not receive any information about _which_ state variable it refers to. There is no "identifier" that is passed to `useState`.

To enable their concise syntax, Hooks **rely on a stable call order on every render of the same component**. This works well in practice because if you follow the rule above ("only call Hooks at the top level"), Hooks will always be called in the same order.

Internally, React holds an array of state pairs for every component. It also maintains the current pair index, which is set to `0` before rendering. Each time you call `useState`, React gives you the next state pair and increments the index.

This example doesn't use React but it gives you an idea of how `useState` works internally:

```JSX
let componentHooks = [];
let currentHookIndex = 0;

// How useState works inside React (simplified).
function useState(initialState) {
  let pair = componentHooks[currentHookIndex];
  if (pair) {
    // This is not the first render,
    // so the state pair already exists.
    // Return it and prepare for next Hook call.
    currentHookIndex++;
    return pair;
  }

  // This is the first time we're rendering,
  // so create a state pair and store it.
  pair = [initialState, setState];

  function setState(nextState) {
    // When the user requests a state change,
    // put the new value into the pair.
    pair[0] = nextState;
    updateDOM();
  }

  // Store the pair for future renders
  // and prepare for the next Hook call.
  componentHooks[currentHookIndex] = pair;
  currentHookIndex++;
  return pair;
}

function Gallery() {
  // Each useState() call will get the next pair.
  const [index, setIndex] = useState(0);
  const [showMore, setShowMore] = useState(false);

  function handleNextClick() {
    setIndex(index + 1);
  }

  function handleMoreClick() {
    setShowMore(!showMore);
  }

  let sculpture = sculptureList[index];
  // This example doesn't use React, so
  // return an output object instead of JSX.
  return {
    onNextClick: handleNextClick,
    onMoreClick: handleMoreClick,
    header: `${sculpture.name} by ${sculpture.artist}`,
    counter: `${index + 1} of ${sculptureList.length}`,
    more: `${showMore ? 'Hide' : 'Show'} details`,
    description: showMore ? sculpture.description : null,
    imageSrc: sculpture.url,
    imageAlt: sculpture.alt
  };
}

function updateDOM() {
  // Reset the current Hook index
  // before rendering the component.
  currentHookIndex = 0;
  let output = Gallery();

  // Update the DOM to match the output.
  // This is the part React does for you.
  nextButton.onclick = output.onNextClick;
  header.textContent = output.header;
  moreButton.onclick = output.onMoreClick;
  moreButton.textContent = output.more;
  image.src = output.imageSrc;
  image.alt = output.imageAlt;
  if (output.description !== null) {
    description.textContent = output.description;
    description.style.display = '';
  } else {
    description.style.display = 'none';
  }
}

let nextButton = document.getElementById('nextButton');
let header = document.getElementById('header');
let moreButton = document.getElementById('moreButton');
let description = document.getElementById('description');
let image = document.getElementById('image');
let sculptureList = [{}];

// Make UI match the initial state.
updateDOM();
```

## State is isolated and private

State is local to a component instance on the screen. In other words, **if you render the same component twice, each copy will have completely isolated state.** Changing one of them will not affect the other.
## Recap 

- Use a state variable when a component needs to “remember” some information between renders.
- State variables are declared by calling the `useState` Hook.
- Hooks are special functions that start with `use`. They let you “hook into” React features like state.
- Hooks might remind you of imports: they need to be called unconditionally. Calling Hooks, including `useState`, is only valid at the top level of a component or another Hook.
- The `useState` Hook returns a pair of values: the current state and the function to update it.
- You can have more than one state variable. Internally, React matches them up by their order.
- State is private to the component. If you render it in two places, each copy gets its own state.