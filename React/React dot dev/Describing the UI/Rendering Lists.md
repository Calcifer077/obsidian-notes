---
title: Rendering Lists
source: https://react.dev/learn/rendering-lists
created: 2026-07-29
tags:
  - react
---
## Rendering data from arrays

Here's a short example of how to generate a list of items from an array:

### 1. Move the data into an array:

```JSX
const people = [
  'Creola Katherine Johnson: mathematician',
  'Mario José Molina-Pasquel Henríquez: chemist',
  'Mohammad Abdus Salam: physicist',
  'Percy Lavon Julian: chemist',
  'Subrahmanyan Chandrasekhar: astrophysicist'
];
```

### 2. Map the `people` members into a new array of JSX nodes, `listItems`:

```JSX
const listItems = people.map(person => <li>{person}</li>);
```

### 3. Return `listItems` from your component wrapped in a `<ul>`:

```JSX
return <ul>{listItems}</ul>;
```

Here is the result:

```JSX
const people = [
  'Creola Katherine Johnson: mathematician',
  'Mario José Molina-Pasquel Henríquez: chemist',
  'Mohammad Abdus Salam: physicist',
  'Percy Lavon Julian: chemist',
  'Subrahmanyan Chandrasekhar: astrophysicist'
];

export default function List() {
  const listItems = people.map(person =>
    <li>{person}</li>
  );
  return <ul>{listItems}</ul>;
}
```

## Filtering arrays of items

Suppose we have the following data:

```JSX
const people = [{
  id: 0,
  name: 'Creola Katherine Johnson',
  profession: 'mathematician',
}, {
  id: 1,
  name: 'Mario José Molina-Pasquel Henríquez',
  profession: 'chemist',
}, {
  id: 2,
  name: 'Mohammad Abdus Salam',
  profession: 'physicist',
}, {
  id: 3,
  name: 'Percy Lavon Julian',
  profession: 'chemist',
}, {
  id: 4,
  name: 'Subrahmanyan Chandrasekhar',
  profession: 'astrophysicist',
}];
```

and we want to render only chemist. We can use JS's `filter()` method to return just those people. 

```JSX
const chemists = people.filter(person =>
  person.profession === 'chemist'
);
```

Now we can render them using `map()` as discussed above. 

## Keeping list items in order with `key`

If we were to run any of the above rendering code, it will show following error in the console:

```
Warning: Each child in a list should have a unique “key” prop.
```

We need to give each array item a `key` - a string or a number that uniquely identifies it among other items in that array:

```JSX
<li key={person.id}>...</li>
```

_Note: JSX elements directly inside a `map()` call always need keys!_

Keys tell React which array item each component corresponds to, so that it can match them up later. This becomes important if your array items can move (e.g. due to sorting), get inserted, or get deleted. A well-chosen `key` helps React infer what exactly has happened, and make the correct updates to the DOM tree.

### Displaying several DOM nodes for each list item

What to do when each item needs to render not one, but several DOM nodes?

The short [`<>...</>` Fragment](https://react.dev/reference/react/Fragment) syntax won’t let you pass a key, so you need to either group them into a single `<div>`, or use the slightly longer and [more explicit `<Fragment>` syntax:](https://react.dev/reference/react/Fragment#rendering-a-list-of-fragments)

```JSX
import { Fragment } from 'react';

// ...

const listItems = people.map(person =>
  <Fragment key={person.id}>
    <h1>{person.name}</h1>
    <p>{person.bio}</p>
  </Fragment>
);
```

## Rules of keys

- **Keys must be unique among siblings**. However, it's okay to use the same keys for JSX nodes in _different_ arrays.
- **Keys must not change** or that defeats their purpose! Don't generate them while rendering.

## Why does React need keys?

Imagine that files on your desktop didn’t have names. Instead, you’d refer to them by their order — the first file, the second file, and so on. You could get used to it, but once you delete a file, it would get confusing. The second file would become the first file, the third file would be the second file, and so on.

File names in a folder and JSX keys in an array serve a similar purpose. They let us uniquely identify an item between its siblings. A well-chosen key provides more information than the position within the array. Even if the _position_ changes due to reordering, the `key` lets React identify the item throughout its lifetime.

### Pitfall 

You might be tempted to use an item’s index in the array as its key. In fact, that’s what React will use if you don’t specify a `key` at all. But the order in which you render items will change over time if an item is inserted, deleted, or if the array gets reordered. Index as a key often leads to subtle and confusing bugs.

Similarly, do not generate keys on the fly, e.g. with `key={Math.random()}`. This will cause keys to never match up between renders, leading to all your components and DOM being recreated every time. Not only is this slow, but it will also lose any user input inside the list items. Instead, use a stable ID based on the data.

Note that your components won’t receive `key` as a prop. It’s only used as a hint by React itself. If your component needs an ID, you have to pass it as a separate prop: `<Profile key={id} userId={id} />`.

## Recap

- How to move data out of components and into data structures like arrays and objects.
- How to generate sets of similar components with JavaScript’s `map()`.
- How to create arrays of filtered items with JavaScript’s `filter()`.
- Why and how to set `key` on each component in a collection so React can keep track of each of them even if their position or data changes.