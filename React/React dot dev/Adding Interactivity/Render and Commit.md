---
title: Render and Commit
source: https://react.dev/learn/render-and-commit
created: 2026-08-02
tags:
  - react
---
Before your components are displayed on screen, they must be rendered by React.

magine that your components are cooks in the kitchen, assembling tasty dishes from ingredients. In this scenario, React is the waiter who puts in requests from customers and brings them their orders. This process of requesting and serving UI has three steps:

1. **Triggering** a render (delivering the guest’s order to the kitchen)
2. **Rendering** the component (preparing the order in the kitchen)
3. **Committing** to the DOM (placing the order on the table)

![](../../../assets/Pasted%20image%2020260802173313.png)

## Step 1: Trigger a render

There are two reasons for a component to render:
1. It's the component's **initial render**.
2. The component's (or one of its ancestors') **state has been updated**.

### Initial Render

When your app starts, you need to trigger the initial render. It is done by calling [`createRoot`](https://react.dev/reference/react-dom/client/createRoot) with the target DOM node, and then calling its `render` method with your component. 

```JSX
import Image from './Image.js';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'))
root.render(<Image />);
```

### Re-renders when state updates

Once the component has been initially rendered, you can trigger further renders by updating its state with the [`set` function.](https://react.dev/reference/react/useState#setstate) Updating your component’s state automatically queues a render. (You can imagine these as a restaurant guest ordering tea, dessert, and all sorts of things after putting in their first order, depending on the state of their thirst or hunger.)

![](../../../assets/Pasted%20image%2020260802173659.png)

## Step 2: React renders your components

After you trigger a render, React calls your components to figure out what to display on screen. **“Rendering” is React calling your components.**

- **On initial render,** React will call the root component.
- **For subsequent renders,** React will call the function component whose state update triggered the render.

This process is recursive: if the updated component returns some other component, React will render that component next and so on until there are no more nested components and React knows exactly what should be displayed on screen.

_Note: Rendering must always be pure._

## Step 3: React commits changes to the DOM

After rendering (calling) your components, React will modify the DOM.

- **For the initial render,** React will use the [`appendChild()`](https://developer.mozilla.org/docs/Web/API/Node/appendChild) DOM API to put all the DOM nodes it has created on screen.
- **For re-renders,** React will apply the minimal necessary operations (calculated while rendering!) to make the DOM match the latest rendering output.

**React only changes the DOM nodes if there's a difference between renders**.

## Epilogue: Browser paint

After rendering is done and React updated the DOM, the browser will repaint the screen.

## Recap 

- Any screen update in a React app happens in three steps:
    1. Trigger
    2. Render
    3. Commit
- You can use Strict Mode to find mistakes in your components
- React does not touch the DOM if the rendering result is the same as last time