---
title: Passing data deeply with context
source: https://react.dev/learn/passing-data-deeply-with-context
created: 2026-08-17
tags:
  - react
---
Usually, you will pass information from a parent component to a child component via props. This can be inconvenient when you have to pass through through many components or if many components in your app need the same information. _Context_ lets the parent component make some information available to any component in the tree below it without explicitly passing through props. 

## Context: an alternative to passing props 

Context lets a parent component provide data to the entire tree below it.

Consider below example:

```JSX
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading level={1}>Title</Heading>
      <Heading level={2}>Heading</Heading>
      <Heading level={3}>Sub-heading</Heading>
      <Heading level={4}>Sub-sub-heading</Heading>
      <Heading level={5}>Sub-sub-sub-heading</Heading>
      <Heading level={6}>Sub-sub-sub-sub-heading</Heading>
    </Section>
  );
}
```

`Section` just renders it children. `Heading` accepts a `level` prop to determine its size.

Let's say you want multiple headings within the same `Section` to always have the same size:

```JSX
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading level={1}>Title</Heading>
      <Section>
        <Heading level={2}>Heading</Heading>
        <Heading level={2}>Heading</Heading>
        <Heading level={2}>Heading</Heading>
        <Section>
          <Heading level={3}>Sub-heading</Heading>
          <Heading level={3}>Sub-heading</Heading>
          <Heading level={3}>Sub-heading</Heading>
          <Section>
            <Heading level={4}>Sub-sub-heading</Heading>
            <Heading level={4}>Sub-sub-heading</Heading>
            <Heading level={4}>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

Currently, you pass the `level` prop to each `Heading` separately:

```JSX
<Section>
  <Heading level={3}>About</Heading>
  <Heading level={3}>Photos</Heading>
  <Heading level={3}>Videos</Heading>
</Section>
```

Wouldn't it be nice if you could pass the `level` prop to the `Section` component instead and remove it from the `Heading`.

```JSX
<Section level={3}>
  <Heading>About</Heading>
  <Heading>Photos</Heading>
  <Heading>Videos</Heading>
</Section>
```

But how can the `Heading` component know the level of its closes `Section`. That would require some way for a child to "ask" for data from somewhere above in the tree.

This is where context comes into play. You can do so in three steps:
1. **Create** a context. (You can call it `LevelContext`, since it’s for the heading level.)
2. **Use** that context from the component that needs the data. (`Heading` will use `LevelContext`.)
3. **Provide** that context from the component that specifies the data. (`Section` will provide `LevelContext`.)

### Step 1. Create a context 

First, you need to create a context. You'll need to export it from a file so that your components can use it:

```JSX
import { createContext } from 'react';

export const LevelContext = createContext(1);
```

The only argument to `createContext` is the default value.

### Step 2: Use the context 

Import the `useContext` hook from react and your context:

```JSX
import { useContext } from 'react';
import { LevelContext } from './file-that-contains-context.js';
```

Currently, the `Heading` component reads `level` from props:

```JSX
export default function Heading({ level, children }) {
  // ...
}
```

Instead, remove the `level` prop and read the value from the context you just imported:

```JSX
export default function Heading({ children }) {
  const level = useContext(LevelContext);
  // ...
}
```

**`useContext` tells React that the `Heading` component wants to read the `LevelContext`.**

Now that the `Heading` component doesn't have a `level` prop, you don't need to pass the level prop to `Heading` in your JSX, instead the parent component `Section` can receive it.

```JSX
<Section level={4}>
  <Heading>Sub-sub-heading</Heading>
  <Heading>Sub-sub-heading</Heading>
  <Heading>Sub-sub-heading</Heading>
</Section>
```

### Step 3: Provide the context 

The `Section` component currently renders its children:

```JSX
export default function Section({ children }) {
  return (
    <section className="section">
      {children}
    </section>
  );
}
```

Wrap them with a context provider to provide the `LevelContext` to them:

```JSX
import { LevelContext } from './LevelContext.js';

export default function Section({ level, children }) {
  return (
    <section className="section">
      <LevelContext value={level}>
        {children}
      </LevelContext>
    </section>
  );
}
```

This tells React: “if any component inside this `<Section>` asks for `LevelContext`, give them this `level`.” The component will use the value of the nearest `<LevelContext>` in the UI tree above it.

## Before you use context 

Just because you need to pass some props several levels deep doesn't mean you should put that information into context.

Here's a few alternatives you should consider before using context:

1. Start by **passing props**. It's not unusual to pass a dozen props down through a dozen components. It may feel like a slog, but it makes it very clear which components use which data.
2. Extract components and **pass JSX as `children`** to them. If you pass some data through many layers of intermediate components that don't use that data (and only pass it further down), this often means that you forgot to extract some components along the way.

## Use cases for context 

- **Theming:** If your app lets the user change its appearance (e.g. dark mode), you can put a context provider at the top of your app, and use that context in components that need to adjust their visual look.
- **Current account:** Many components might need to know the currently logged in user. Putting it in context makes it convenient to read it anywhere in the tree. Some apps also let you operate multiple accounts at the same time (e.g. to leave a comment as a different user). In those cases, it can be convenient to wrap a part of the UI into a nested provider with a different current account value.
- **Routing:** Most routing solutions use context internally to hold the current route. This is how every link “knows” whether it’s active or not. If you build your own router, you might want to do it too.

Context is not limited to static values. If you pass a different value on the next render, React will update all the components reading it below! This is why context is often used in combination with state.

## Recap 

- Context lets a component provide some information to the entire tree below it.
- To pass context:
    1. Create and export it with `export const MyContext = createContext(defaultValue)`.
    2. Pass it to the `useContext(MyContext)` Hook to read it in any child component, no matter how deep.
    3. Wrap children into `<MyContext value={...}>` to provide it from a parent.
- Context passes through any components in the middle.
- Context lets you write components that “adapt to their surroundings”.
- Before you use context, try passing props or passing JSX as `children`.