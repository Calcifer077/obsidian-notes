---
title: Preserving and Resetting state
source: https://react.dev/learn/preserving-and-resetting-state
created: 2026-08-13
tags:
  - react
---
State is isolated between components. React keeps track of which state belongs to which component based on their place in the UI tree. 

## State is tied to a position in the render tree 

React builds render trees for the component structure in your UI. 

When you give a component state, you might think the state "lives" inside the component. But the state is actually held inside react. React associates each piece of state it's holding with the correct component by where that component sits in the render tree. 

Here, there is only one `<Counter />` JSX tag, but it's rendered at two different positions:

```JSX
import { useState } from 'react';

export default function App() {
  const counter = <Counter />;
  return (
    <div>
      {counter}
      {counter}
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

![](../../../assets/Pasted%20image%2020260813153700.png)

Both `<Counter />` are independent of each other, updating `count` in one won't affect the other's state. 

React will keep the state around for as long as you render the same component at the same position in the tree. If you remove a component from the tree, its state will be destroyed. 

```JSX
import { useState } from 'react';

export default function App() {
  const [showB, setShowB] = useState(true);
  return (
    <div>
      <Counter />
      {showB && <Counter />}
      <label>
        <input
          type="checkbox"
          checked={showB}
          onChange={e => {
            setShowB(e.target.checked)
          }}
        />
        Render the second counter
      </label>
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

Above we have a check box which can remove or add a `<Counter />` component. The moment you remove it from the tree, its state will be destroyed and when you check the checkbox again, a new `<Counter />` will be displayed with new state. 

**React preserves a component’s state for as long as it’s being rendered at its position in the UI tree.** If it gets removed, or a different component gets rendered at the same position, React discards its state.

## Same component at the same position preserves state 

In this example, there are two different `<Counter />` tags:

```JSX
import { useState } from 'react';

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy ? (
        <Counter isFancy={true} />
      ) : (
        <Counter isFancy={false} />
      )}
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={e => {
            setIsFancy(e.target.checked)
          }}
        />
        Use fancy styling
      </label>
    </div>
  );
}

function Counter({ isFancy }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }
  if (isFancy) {
    className += ' fancy';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

When you tick or clear the checkbox, the counter state does not get reset. Whether `isFancy` is `true` or `false`, you always have a `<Counter />` as the first child of the `div` returned from the root `App` component:

![](../../../assets/Pasted%20image%2020260813154315.png)

It's the same component at the same position, so from React's perspective, it's the same counter. 

## Different components at the same position reset state 

If you render different components at the same position in the UI tree, state will be reset. 

In this example, ticking the checkbox will replace `<Counter>` with a `<p>`:

```JSX
import { useState } from 'react';

export default function App() {
  const [isPaused, setIsPaused] = useState(false);
  return (
    <div>
      {isPaused ? (
        <p>See you later!</p>
      ) : (
        <Counter />
      )}
      <label>
        <input
          type="checkbox"
          checked={isPaused}
          onChange={e => {
            setIsPaused(e.target.checked)
          }}
        />
        Take a break
      </label>
    </div>
  );
}
```

Here, you switch between _different_ component types at the same position. Initially, the first child of the `<div>` contained a `Counter`. But when you swapped in a `p`, React removed the `Counter` from the UI tree and destroyed its state.

**When you render a different component in the same position, it resets the state of its entire subtree**. 

```JSX
import { useState } from 'react';

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy ? (
        <div>
          <Counter isFancy={true} />
        </div>
      ) : (
        <section>
          <Counter isFancy={false} />
        </section>
      )}
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={e => {
            setIsFancy(e.target.checked)
          }}
        />
        Use fancy styling
      </label>
    </div>
  );
}
```

The counter state gets reset when you click the checkbox. Although you render a `Counter`, the first child of the `div` changes from a `section` to a `div`. When the child `section` was removed from the DOM, the whole tree below it (including the `Counter` and its state) was destroyed as well.

![](../../../assets/Pasted%20image%2020260813154831.png)

As a rule of thumb, **if you want to preserver the state between re-renders, the structure of your tree needs to "match up"** from one render to another. If the structure is different, the state gets destroyed because react destroys state when it removes a component from the tree.

## Resetting state at the same position 

By default, react preserves state of a component while it stays at the same position. Usually, this is exactly what you want, so it makes sense as the default behavior. But sometimes, you may want to reset a component's state. Consider the following example:

```JSX
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA ? (
        <Counter person="Taylor" />
      ) : (
        <Counter person="Sarah" />
      )}
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        Next player!
      </button>
    </div>
  );
}
```

Currently, if you change the player, the score is preserved. The two `Coutner`'s appear in the same position, so react sees them as the same `Counter` whose `person` prop has changed. 

But conceptually, in this app they should be two separate counters. They might appear in the same place in the UI, but one is a counter for Taylor, and another is a counter for Sarah.

There are two ways to reset state when switching between them:

1. Render components in different positions
2. Give each component an explicit identity with `key`

### Option 1: Rendering a component in different positions 

We can render them at different positions to make them independent.

```JSX
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA &&
        <Counter person="Taylor" />
      }
      {!isPlayerA &&
        <Counter person="Sarah" />
      }
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        Next player!
      </button>
    </div>
  );
}
```

- Initially, `isPlayerA` is `true`. So the first position contains `Counter` state, and the second one is empty.
- When you click the “Next player” button the first position clears but the second one now contains a `Counter`.

![](../../../assets/Pasted%20image%2020260813155514.png)

Each `Counter`'s state gets destroyed each time it's removed from the DOM.

### Option 2: Resetting state with a key 

You can use keys to make react distinguish between any components. By default, React uses order within the parent (“first counter”, “second counter”) to discern between components. But keys let you tell React that this is not just a _first_ counter, or a _second_ counter, but a specific counter, for example, _Taylor’s_ counter. This way, React will know _Taylor’s_ counter wherever it appears in the tree!

```JSX
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA ? (
        <Counter key="Taylor" person="Taylor" />
      ) : (
        <Counter key="Sarah" person="Sarah" />
      )}
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        Next player!
      </button>
    </div>
  );
}
```

Switching between the two `Counter` reset their state due to presence of `key`.

Specifying a `key` tells react to use the `key` itself as part of the position, instead of their order within the parent. This is why, even though you render them in the same place in JSX, React sees them as two different counters, and so they will never share state.
## Recap 

- React keeps state for as long as the same component is rendered at the same position.
- State is not kept in JSX tags. It’s associated with the tree position in which you put that JSX.
- You can force a subtree to reset its state by giving it a different key.
- Don’t nest component definitions, or you’ll reset state by accident.