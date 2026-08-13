---
title: Choosing the State Structure
source: https://react.dev/learn/choosing-the-state-structure
created: 2026-08-12
tags:
  - react
---
## Principles for structuring state 

There are a few principles that can guide you to make better choices when deciding how many state variables to use and what the shape of their data should be.

1. **Group related state.** If you always update two or more state variables at the same time, consider merging them into a single state variable.
2. **Avoid contradictions in state.** Try to avoid state structures when many pieces of state may disagree with each other.
3. **Avoid redundant state.** If you can calculate something don't put it in state. 
4. **Avoid duplication in state.** When the same data is duplicated between multiple state variables, or within nested objects, it is difficult to keep them in sync. Reduce duplication when you can.
5. **Avoid deeply nested state.** Deeply hierarchical state is not very convenient to update. When possible, prefer to structure state in a flat way.

## Group related state 

In some cases you might be unsure between using single or multiple state variables.

```JSX
const [x, setX] = useState(0);
const [y, setY] = useState(0);

// OR

const [position, setPosition] = useState({ x: 0, y: 0 });
```

Technically, you can use either. But if some two state variables always change together, it might be a good idea to unify them into a single state variable. 

Another case where you'll group data into an object or an array is when you don't know how many pieces of state you'll need. For example, it’s helpful when you have a form where the user can add custom fields.

## Avoid contradictions in state 

Suppose you have a form, which have two state variables `isSending` (is the user sending some message?) and `isSent` (have the process been completed?) regarding messages. We can work around this, but may hit a wall when both `isSending` and `isSent` becomes true at the same time (It's kind of impossible but still). To avoid room for error we can simply remove both and use something much simpler.

We can use `status` state variable which can have any of the three values `typing`, `sending`, and `sent`. It does the work of both `isSending` and `isSent` and can never create any complications like above. 

## Avoid redundant state 

If you can calculate some information from the component’s props or its existing state variables during rendering, you **should not** put that information into that component’s state.

```JSX
const [firstName, setFirstName] = useState('');
const [lastName, setLastName] = useState('');
const [fullName, setFullName] = useState('');
```

Above component has three state variables. `firstName`, `lastName` and `fullName`. However, `fullName` is redundant. We can always calculate `fullName` from `firstName` and `lastName` during render, so we can remove it from state.

```JSX
const [firstName, setFirstName] = useState('');
const [lastName, setLastName] = useState('');

const fullName = firstName + ' ' + lastName;
```

### Don't mirror props in state 

A common example of redundant state is code like this:

```JSX
function Message({ messageColor }) {
  const [color, setColor] = useState(messageColor);
```

Here, a `color` state variable is initialized to the `messageColor` prop. The problem is that if the parent component passes a different value of `messageColor` later, the `color` state _variable_ would not be updated! The state is only initialized during the first render. 

**Why above thing happens?**

When react renders your component it keeps a persistent record somewhere outside your function (refer to further reading), think of it as a slot in memory tied to that specific component in the tree. Each call to `useState` in your component corresponds to one of these slot, in the order they're called.

The first time your component mounts:

```JSX
const [color, setColor] = useState(messageColor);
```

React sees that there is no "slot" for this `useState` call, so it creates one, and initializes it with whatever you passed in (`messageColor`). From now on, that slot holds that value and for every subsequent render, may it be due to parent re rendering, state update etc. React will see that it already has a "slot" for this state and will just return that. It wouldn't update it unless you update it using state setter function.

>Note: If you do need to update state in child, use state setter function from parent itself. 

## Avoid duplication in state 

Below example demonstrates a menu list component that lets you choose a single travel snack out of several:

```JSX
const initialItems = [
  { title: 'pretzels', id: 0 },
  { title: 'crispy seaweed', id: 1 },
  { title: 'granola bar', id: 2 },
];

const [items, setItems] = useState(initialItems);
const [selectedItem, setSelectedItem] = useState(items[0]);
```

> Note: visit source for full version of code.

If the user were to select any items we would have to do the hassle of keeping them in sync. We could do selection of items easily but it would be a problem to maintain integrity. In the below code we allow the user to update items but we the `selectedItem` would go out of sync.

```JSX
 function handleItemChange(id, e) {
    setItems(items.map(item => {
      if (item.id === id) {
        return {
          ...item,
          title: e.target.value,
        };
      } else {
        return item;
      }
    }));
  }
```

An easier approach would be to derive the state and avoid duplication. 

```JSX
const [selectedId, setSelectedId] = useState(0);

const selectedItem = items.find(item =>
    item.id === selectedId
);
```

Now, even if the selected item is updated, the state will remain consistent.

## Avoid deeply nested state 

If you have too deeply nested state it would be be difficult to update it, Why? Because you would have to copy the entire object and update it and than set it.

**If the state is too nested to update easily, consider making it "flat".**

## Recap 

- If two state variables always update together, consider merging them into one.
- Choose your state variables carefully to avoid creating “impossible” states.
- Structure your state in a way that reduces the chances that you’ll make a mistake updating it.
- Avoid redundant and duplicate state so that you don’t need to keep it in sync.
- Don’t put props _into_ state unless you specifically want to prevent updates.
- For UI patterns like selection, keep ID or index in state instead of the object itself.
- If updating deeply nested state is complicated, try flattening it.
## Further reading 

- [More reasoning for **Don't mirror props in state**](https://react.dev/learn/state-a-components-memory#how-does-react-know-which-state-to-return)