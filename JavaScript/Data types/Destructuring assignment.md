---
title: Destructuring assignment
source: https://javascript.info/destructuring-assignment
created: 2026-06-22
tags:
  - javascript
description: Contains destructuring of arrays and objects
---
_Destructuring assignment_ is a special syntax that allows us to "unpack" arrays or objects into a bunch of variables. 

## Array destructuring

Here’s an example of how an array is destructured into variables:
```js
// we have an array with a name and surname
let arr = ["John", "Smith"]

// destructuring assignment
// sets firstName = arr[0]
// and surname = arr[1]
let [firstName, surname] = arr;

alert(firstName); // John
alert(surname);  // Smith
```

Now we can work with variables instead of array members.

It looks great when combined with `split` or other array-returning methods:

```js
let [firstName, surname] = "John Smith".split(' ');
alert(firstName); // John
alert(surname);  // Smith
```

#### “Destructuring” does not mean “destructive”.
It’s called “destructuring assignment,” because it “destructurizes” by copying items into variables. However, the array itself is not modified.

It’s just a shorter way to write:
```js
// let [firstName, surname] = arr;
let firstName = arr[0];
let surname = arr[1];
```

#### Ignore elements using commas
Unwanted elements of the array can also be thrown away via an extra comma:
```js
// second element is not needed
let [firstName, , title] = ["Julius", "Caesar", "Consul", "of the Roman Republic"];

alert( title ); // Consul
```

In the code above, the second element of the array is skipped, the third one is assigned to `title`, and the rest of the array items are also skipped (as there are no variables for them).

#### Works with any iterable on the right-side
We can use it with any iterable, not only arrays:
```js
let [a, b, c] = "abc"; // ["a", "b", "c"]
let [one, two, three] = new Set([1, 2, 3]);
```

This works, because internally a destructuring assignment works by iterating over the right value. It's kind of syntax sugar for calling `for..of` over the value to the right of `=` and assigning the values.

## The rest `'...'`

Usually, if the array is longer than the list at the left, the "extra" items are omitted. If we'd like to gather all that follows - we can add one more parameter that gets "the rest" using three dots `"..."`:

```js
let [name1, name2, ...rest] = ["Julius", "Caesar", "Consul", "of the Roman Republic"];

// rest is an array of items, starting from the 3rd one
alert(rest[0]); // Consul
alert(rest[1]); // of the Roman Republic
alert(rest.length); // 2
```

The value of `rest` is the array of the remaining array elements.

## Default values

If the array is shorter than the list of variables on the left, there will be no errors. Absent values are considered undefined:
```js
let [firstName, surname] = [];

alert(firstName); // undefined
alert(surname); // undefined
```

If we want a "default" value to replace the missing one, we can provide it using `=`:
```js
// default values
let [name = "Guest", surname = "Anonymous"] = ["Julius"];

alert(name);    // Julius (from array)
alert(surname); // Anonymous (default used)
```

Default values can be more complex expressions or even function calls. They are evaluated only if the value is not provided.

```js
// runs only prompt for surname
let [name = prompt('name?'), surname = prompt('surname?')] = ["Julius"];

alert(name);    // Julius (from array)
alert(surname); // whatever prompt gets
```

Note: the `prompt` will run only  for the missing value (`surname`).

## Object destructuring

The destructuring assignment also works with objects. The basic syntax is:
```js
let {var1, var2} = {var1:..., var2:...}
```

We should have an existing object on the right side, that we want to split into variables. The left side contains an object-like "pattern" for corresponding properties. In the simplest case, that's a list of variable names in `{...}`.

For instance:
```js
let options = {
  title: "Menu",
  width: 100,
  height: 200
};

let {title, width, height} = options;

alert(title);  // Menu
alert(width);  // 100
alert(height); // 200
```

Properties `options.title`, `options.width` and `options.height` are assigned to the corresponding variables. 

**The order does not matter.**

The pattern on the left side may be more complex and specify the mapping between properties and variables.

If we want to assign a property to a variable with another name, for instance, make `options.width` go into the variable named `w`, then we can set the variable name using a colon:

```js
let options = {
  title: "Menu",
  width: 100,
  height: 200
};

// { sourceProperty: targetVariable }
let {width: w, height: h, title} = options;

// width -> w
// height -> h
// title -> title

alert(title);  // Menu
alert(w);      // 100
alert(h);      // 200
```

The colon shows “what : goes where”. In the example above the property `width` goes to `w`, property `height` goes to `h`, and `title` is assigned to the same name.

For potentially missing properties we can set default values using `"="`, like this:
```js
let options = {
  title: "Menu"
};

let {width = 100, height = 200, title} = options;

alert(title);  // Menu
alert(width);  // 100
alert(height); // 200
```

Just like with arrays or function parameters, default values can be any expressions or even function calls. They will be evaluated if the value is not provided.

We can also combine both the colon and equality:
```js
let options = {
  title: "Menu"
};

let {width: w = 100, height: h = 200, title} = options;

alert(title);  // Menu
alert(w);      // 100
alert(h);      // 200
```

## The rest pattern `"..."`

We can use the rest pattern, just like we did with array. It's not supported by some older browsers (use Babel to polyfill it).

It looks like this:
```js
let options = {
  title: "Menu",
  height: 200,
  width: 100
};

// title = property named title
// rest = object with the rest of properties
let {title, ...rest} = options;

// now title="Menu", rest={height: 200, width: 100}
alert(rest.height);  // 200
alert(rest.width);   // 100
```

#### Gotcha if there's no `let`
In the examples above variables were declared right in the assignment: `let {…} = {…}`. Of course, we could use existing variables too, without `let`. But there’s a catch.

This won't work:
```js
let title, width, height;

// error in this line
{title, width, height} = {title: "Menu", width: 200, height: 100};
```

The problem is that JavaScript treats `{...}` in the main code flow (not inside another expression) as a code block. Such code blocks can be used to group statements, like this:
```js
{
  // a code block
  let message = "Hello";
  // ...
  alert( message );
}
```

So here JavaScript assumes that we have a code block, that's why there's an error. We want destructuring instead.

To show JavaScript that it's not a code block, we can wrap the expression in parentheses `(...)`:

```js
let title, width, height;

// okay now
({title, width, height} = {title: "Menu", width: 200, height: 100});

alert( title ); // Menu
```

## Nested destructuring

If an object or an array contains other nested objects and arrays, we can use more complex left-side patterns to extract deeper portions. 

In the code below `options` has another object in the property `size` and an array in the property `items`. The pattern on the left side of the assignment has the same structure to extract values from them:
```js
let options = {
  size: {
    width: 100,
    height: 200
  },
  items: ["Cake", "Donut"],
  extra: true
};

// destructuring assignment split in multiple lines for clarity
let {
  size: { // put size here
    width,
    height
  },
  items: [item1, item2], // assign items here
  title = "Menu" // not present in the object (default value is used)
} = options;

alert(title);  // Menu
alert(width);  // 100
alert(height); // 200
alert(item1);  // Cake
alert(item2);  // Donut
```

All properties of `options` object except `extra` which is absent in the left part, are assigned to corresponding variables:

![](../../assets/Pasted%20image%2020260622160227.png)

Finally, we have `width`, `height`, `item1`, `item2` and `title` from the default value. Note that there are no variables for `size` and `items`, as we take their content instead.

## Smart function parameters

There are times when a function has many parameters, most of which are optional. That’s especially true for user interfaces. Imagine a function that creates a menu. It may have a width, a height, a title, an item list and so on.

Here’s a bad way to write such a function:
```js
function showMenu(title = "Untitled", width = 200, height = 100, items = []) {
  // ...
}
```

In real-life, the problem is how to remember the order of arguments. Usually, IDEs try to help us, especially if the code is well-documented, but still… Another problem is how to call a function when most parameters are ok by default.

Like this?
```js
// undefined where default values are fine
showMenu("My Menu", undefined, undefined, ["Item1", "Item2"])
```

That's ugly. And becomes unreadable when we deal with more parameters. We can avoid this using destructuring.

We can pass parameters as an object, and the function immediately destructurizes them into variables:
```js
// we pass object to function
let options = {
  title: "My menu",
  items: ["Item1", "Item2"]
};

// ...and it immediately expands it to variables
function showMenu({title = "Untitled", width = 200, height = 100, items = []}) {
  // title, items – taken from options,
  // width, height – defaults used
  alert( `${title} ${width} ${height}` ); // My Menu 200 100
  alert( items ); // Item1, Item2
}

showMenu(options);
```

We can also use more complex destructuring with nested objects and colon mappings:
```js
let options = {
  title: "My menu",
  items: ["Item1", "Item2"]
};

function showMenu({
  title = "Untitled",
  width: w = 100,  // width goes to w
  height: h = 200, // height goes to h
  items: [item1, item2] // items first element goes to item1, second to item2
}) {
  alert( `${title} ${w} ${h}` ); // My Menu 100 200
  alert( item1 ); // Item1
  alert( item2 ); // Item2
}

showMenu(options);
```

The full syntax is the same as for a destructuring assignment:
```js
function({
  incomingProperty: varName = defaultValue
  ...
})
```

Then, for an object of parameters, there will be a variable `varName` for the property `incomingProperty`, with `defaultValue` by default.

Please note that such destructuring assumes that `showMenu()` does have an argument. If we want all values by default, then we should specify an empty object:
```js
showMenu({}); // ok, all values are default

showMenu(); // this would give an error
```

We can fix this by making `{}` the default value for the whole object of parameters:
```js
function showMenu({ title = "Menu", width = 100, height = 200 } = {}) {
  alert( `${title} ${width} ${height}` );
}

showMenu(); // Menu 100 200
```

In the code above, the whole arguments object is `{}` by default, so there's always something to destructurize.

## Summary

- Destructuring assignment allows for instantly mapping an object or array onto many variables.

- The full object syntax:
```js
let {prop : varName = defaultValue, ...rest} = object
```

This means that property `prop` should go into the variable `varName` and, if no such property exists, then the `default` value should be used.

Object properties that have no mapping are copied to the `rest` object.

- The full array syntax:
```js
let [item1 = defaultValue, item2, ...rest] = array
```

The first item goes to `item1`; the second goes into `item2`, and all the rest makes the array `rest`.

- It’s possible to extract data from nested arrays/objects, for that the left side must have the same structure as the right one.
