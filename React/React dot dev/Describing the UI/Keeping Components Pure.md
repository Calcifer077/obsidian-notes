---
title: Keeping Components Pure
source: https://react.dev/learn/keeping-components-pure
created: 2026-07-30
tags:
  - react
---
Pure functions only perform a calculation and nothing more. By strictly only writing your components as pure functions, you can avoid an entire class of bugs.

## Purity: Components as formulas

In computer science (and especially the world of functional programming), [a pure function](https://wikipedia.org/wiki/Pure_function) is a function with the following characteristics:

- **It minds its own business.** It does not change any objects or variables that existed before it was called.
- **Same inputs, same output.** Given the same inputs, a pure function should always return the same result.

A simple example would like this:

```js
function double(number) {
  return 2 * number;
}
```

In the above example, `double` is a **pure function.** If you pass it `3`, it will return `6`. Always.

React is designed around this concept. **React assumes that every component you write is a pure function.** This means that React components you write must always return the same JSX given the same inputs:

## Side Effects: (un)intended consequences

React's rendering process must always be pure. Components should only return their JSX, and not change any objects or variables _that existed before rendering_ - that would make them impure.

Here is a component that breaks this rule:

```JSX
let guest = 0;

function Cup() {
  // Bad: changing a preexisting variable!
  guest = guest + 1;
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup />
      <Cup />
      <Cup />
    </>
  );
}
```

This component is reading and writing a `guest` variable declared outside of it. This means that **calling this component multiple times will produce different JSX!** And what’s more, if _other_ components read `guest`, they will produce different JSX, too, depending on when they were rendered!

You can fix this component by [Passing `guest` as a prop instead](Passing%20Props%20to%20a%20Component.md):

```JSX
function Cup({ guest }) {
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup guest={1} />
      <Cup guest={2} />
      <Cup guest={3} />
    </>
  );
}
```

Now your component is pure, as the JSX it returns only depends on the `guest` prop.

_Note: You can detect impure calculations using StrictMode. StrictMode renders each component in development twice and checks the two results and compares them and helps us find components that break these rules._

## Local mutation: Your component's little secret

In the above example, the problem was that the component changed a _preexisting_ variable while rendering. This is often called a **“mutation”**. Pure functions don’t mutate variables _outside of the function’s scope or objects that were created before the call_ - that makes them impure!

However, **it’s completely fine to change variables and objects that you’ve _just_ created while rendering.** In this example, you create an `[]` array, assign it to a `cups` variable, and then `push` a dozen cups into it:

```JSX
function Cup({ guest }) {
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaGathering() {
  const cups = [];
  for (let i = 1; i <= 12; i++) {
    cups.push(<Cup key={i} guest={i} />);
  }
  return cups;
}
```

## Where you can cause side effects

In React, **side effects usually belong inside [event handlers.](https://react.dev/learn/responding-to-events)** Event handlers are functions that React runs when you perform some action—for example, when you click a button. Even though event handlers are defined _inside_ your component, they don’t run _during_ rendering! **So event handlers don’t need to be pure.**

If you’ve exhausted all other options and can’t find the right event handler for your side effect, you can still attach it to your returned JSX with a [`useEffect`](https://react.dev/reference/react/useEffect) call in your component. This tells React to execute it later, after rendering, when side effects are allowed. **However, this approach should be your last resort.**

### Why does react care about purity?

Writing pure functions takes some habit and discipline. But it also unlocks marvelous opportunities:

- Your components could run in a different environment—for example, on the server! Since they return the same result for the same inputs, one component can serve many user requests.
- You can improve performance by [skipping rendering](https://react.dev/reference/react/memo) components whose inputs have not changed. This is safe because pure functions always return the same results, so they are safe to cache.
- If some data changes in the middle of rendering a deep component tree, React can restart rendering without wasting time to finish the outdated render. Purity makes it safe to stop calculating at any time.

## Recap

- A component must be pure, meaning:
    - **It minds its own business.** It should not change any objects or variables that existed before rendering.
    - **Same inputs, same output.** Given the same inputs, a component should always return the same JSX.
- Rendering can happen at any time, so components should not depend on each others’ rendering sequence.
- You should not mutate any of the inputs that your components use for rendering. That includes props, state, and context. To update the screen, [“set” state](https://react.dev/learn/state-a-components-memory) instead of mutating preexisting objects.
- Strive to express your component’s logic in the JSX you return. When you need to “change things”, you’ll usually want to do it in an event handler. As a last resort, you can `useEffect`.