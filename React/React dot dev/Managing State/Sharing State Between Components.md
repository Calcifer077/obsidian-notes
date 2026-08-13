---
title: Sharing State Between Components
source: https://react.dev/learn/sharing-state-between-components
created: 2026-08-13
tags:
  - react
---
Sometimes, you want the state of two components to always change together. To do it, remove state from both of them, move it to their closest common parent, and than pass it down to them via props.

## Lifting state up by example 

In this example, a parent `Accordian` component renders two separate `Panel`'s .

Each `Panel` component has a boolean `isActive` state that determines whether its content is visible.

```JSX
import { useState } from 'react';

function Panel({ title, children }) {
  const [isActive, setIsActive] = useState(false);
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={() => setIsActive(true)}>
          Show
        </button>
      )}
    </section>
  );
}

export default function Accordion() {
  return (
    <>
      <h2>Almaty, Kazakhstan</h2>
      <Panel title="About">
        // some text 
      </Panel>
      <Panel title="Etymology">
        // some text
      </Panel>
    </>
  );
}
```

Pressing one panel's button does not affect the other panel, they are independent. 

![](../../../assets/Pasted%20image%2020260813151423.png)

But now let's say we want to change it so that only one panel is expanded at any given time.

To coordinate these two panels, you need to "lift their state up" to a parent component in three steps:

1. Remove state from the child components
2. Pass hardcoded data from the common parent 
3. Add state to the common parent and pass it down together with the event handlers 

This will allow the `Accordian` component to coordinate both `Panel`'s and only expand one at a time.

### Controlled and uncontrolled components 

A component with some local state is called a "uncontrolled" component, whereas a component is "controlled" when the important information in it is driven by props rather than its own local state. 

## Recap 

- When you want to coordinate two components, move their state to their common parent.
- Then pass the information down through props from their common parent.
- Finally, pass the event handlers down so that the children can change the parent’s state.
- It’s useful to consider components as “controlled” (driven by props) or “uncontrolled” (driven by state).