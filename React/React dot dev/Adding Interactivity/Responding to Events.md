---
title: Responding to Events
source: https://react.dev/learn/responding-to-events
created: 2026-08-02
tags:
  - react
---
React lets you add event handlers to your JSX. Event handlers are your own functions that will be triggered in response to interactions like clicking, hovering, focusing form inputs, and so on.

## Adding event handlers

To add an event handler, you will first define a function and then [pass it as a prop](../Describing%20the%20UI/Passing%20Props%20to%20a%20Component.md) to the appropriate JSX tag.

Here's a simple example:

```JSX
export default function Button() {
  function handleClick() {
    alert('You clicked me!');
  }

  return (
    <button onClick={handleClick}>
      Click me
    </button>
  );
}
```

You defined the `handleClick` function and then [pass it as a prop](../Describing%20the%20UI/Passing%20Props%20to%20a%20Component.md)  to `<button>`. `handleClick` is an **event handler.** Event handler functions:

- Are usually defined _inside_ your components.
- Have names that start with `handle`, followed by the name of the event.

By convention, it is common to name event handlers as `handle` followed by the event name. You’ll often see `onClick={handleClick}`, `onMouseEnter={handleMouseEnter}`, and so on.

You can also use inline event handlers.

_Note: Functions passed to event handlers must be passed, not called._

_Note: You can use `e.stopPropagation` to stop propagation in bubbling phase, and you can use `e.preventDefault` to prevent default behaviour of any HTML element._

## Can event handlers have side effects?

Yes, Event handlers are the best place for side effects.

Unlike rendering functions, event handlers don't need to be pure, so it's a great place to change something - for example, change an input's value in response to typing, or change a list in response to a button press.

## Recap

- You can handle events by passing a function as a prop to an element like `<button>`.
- Event handlers must be passed, **not called!** `onClick={handleClick}`, not `onClick={handleClick()}`.
- You can define an event handler function separately or inline.
- Event handlers are defined inside a component, so they can access props.
- You can declare an event handler in a parent and pass it as a prop to a child.
- You can define your own event handler props with application-specific names.
- Events propagate upwards. Call `e.stopPropagation()` on the first argument to prevent that.
- Events may have unwanted default browser behaviour. Call `e.preventDefault()` to prevent that.
- Explicitly calling an event handler prop from a child handler is a good alternative to propagation.