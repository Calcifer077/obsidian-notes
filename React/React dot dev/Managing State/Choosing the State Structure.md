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