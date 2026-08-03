---
title: Updating Arrays in State
source: https://react.dev/learn/updating-arrays-in-state
created: 2026-08-03
tags:
  - react
---
Arrays are mutable in JS, but you should treat them as immutable when you store them in state. Just like with objects, when you want to update an array stored in state, you need to create a new one (or make a copy of an existing one), and then set state to use the new array.

## Updating arrays without mutation

In JS, arrays are just another kind of object. [Like with objects](Updating%20Objects%20in%20State.md), you should treat arrays in React state as read-only. This means that you shouldn't reassign items inside an array like `arr[0] = 'bird'`, and you also shouldn't use methods that mutate the array, such as `push()` and `pop()`. 

Instead, every time you want to update an array, you'll want to pass a new array to your state setting function. To do that, you can create a new array from the original array in your state by calling its non-mutation methods like `filter()` and `map()`. Then you can set your state to the resulting new array.


|           | avoid (mutates the array)           | prefer (returns a new array)                                                                                          |
| --------- | ----------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| adding    | `push`, `unshift`                   | `concat`, `[...arr]` spread syntax ([example](https://react.dev/learn/updating-arrays-in-state#adding-to-an-array))\| |
| removing  | `pop`, `shift`, `splice`            | `filter`, `slice` ([example](https://react.dev/learn/updating-arrays-in-state#removing-from-an-array))                |
| replacing | `splice`, `arr[i] = ...` assignment | `map` ([example](https://react.dev/learn/updating-arrays-in-state#replacing-items-in-an-array))                       |
| sorting   | `reverse`, `sort`                   | copy the array first ([example](https://react.dev/learn/updating-arrays-in-state#making-other-changes-to-an-array))   |

Alternatively, you can [use Immer](https://react.dev/learn/updating-arrays-in-state#write-concise-update-logic-with-immer) which lets you use methods from both columns.

### Adding to an array

Instead of `push` which will mutate the array, use spread syntax:

```JSX
setArtists( // Replace the state
  [ // with a new array
    ...artists, // that contains all the old items
    { id: nextId++, name: name } // and one new item at the end
  ]
);
```

### Removing from an array

To remove an element from a array, simply filter it out using `filter`.

```JSX
setArtists(
  artists.filter(a => a.id !== artist.id)
);
```

Here, `artists.filter(a => a.id !== artist.id)` means "create an array that consists of those `artists` whose IDs are different from `artist.id`".  `filter` does not modify original array but returns a new one.

## Transforming an array

If you want to change some or all items of the array, you can use `map()` to create a new array. The function you will pass to `map` can decide what to with each item.

```JSX
 let initialShapes = [
  { id: 0, type: 'circle', x: 50, y: 100 },
  { id: 1, type: 'square', x: 150, y: 100 },
  { id: 2, type: 'circle', x: 250, y: 100 },
];
 
 const [shapes, setShapes] = useState(
    initialShapes
  );

function handleClick() {
    const nextShapes = shapes.map(shape => {
      if (shape.type === 'square') {
        // No change
        return shape;
      } else {
        // Return a new circle 50px below
        return {
          ...shape,
          y: shape.y + 50,
        };
      }
    });
    // Re-render with the new array
    setShapes(nextShapes);
}
```

_Note: above example may not make much sense, it just shows that you can use `map` to transform a array._

_Note: You can also use `map` to replace items in an array._

### Inserting into an array

Sometimes, you may want to insert an item at a particular position that’s neither at the beginning nor at the end. To do this, you can use the `...` array spread syntax together with the `slice()` method. The `slice()` method lets you cut a “slice” of the array. To insert an item, you will create an array that spreads the slice _before_ the insertion point, then the new item, and then the rest of the original array.

In this example, the insert button always inserts at the index `1`:

```JSX
import { useState } from 'react';

let nextId = 3;
const initialArtists = [
  { id: 0, name: 'Marta Colvin Andrade' },
  { id: 1, name: 'Lamidi Olonade Fakeye'},
  { id: 2, name: 'Louise Nevelson'},
];

export default function List() {
  const [name, setName] = useState('');
  const [artists, setArtists] = useState(
    initialArtists
  );

  function handleClick() {
    const insertAt = 1; // Could be any index
    const nextArtists = [
      // Items before the insertion point:
      ...artists.slice(0, insertAt),
      // New item:
      { id: nextId++, name: name },
      // Items after the insertion point:
      ...artists.slice(insertAt)
    ];
    setArtists(nextArtists);
    setName('');
  }

  return ();
}
```

_Note: If you can't achieve your objective using the above mentioned functions like if you wanted to do `reverse()` or `sort()`, the best alternative is to copy the array using spread syntax, and just mutate that array._

## Updating objects inside arrays

Same methods as above can be used here itself.

## Write concise update logic with Immer

Updating nested arrays without mutation can get a little bit repetitive. [Just as with objects](https://react.dev/learn/updating-objects-in-state#write-concise-update-logic-with-immer):

- Generally, you shouldn’t need to update state more than a couple of levels deep. If your state objects are very deep, you might want to [restructure them differently](https://react.dev/learn/choosing-the-state-structure#avoid-deeply-nested-state) so that they are flat.
- If you don’t want to change your state structure, you might prefer to use [Immer](https://github.com/immerjs/use-immer), which lets you write using the convenient but mutating syntax and takes care of producing the copies for you.

## Recap

- You can put arrays into state, but you can’t change them.
- Instead of mutating an array, create a _new_ version of it, and update the state to it.
- You can use the `[...arr, newItem]` array spread syntax to create arrays with new items.
- You can use `filter()` and `map()` to create new arrays with filtered or transformed items.
- You can use Immer to keep your code concise.