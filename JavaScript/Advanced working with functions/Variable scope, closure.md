---
title: Variable scope, closure
source: https://javascript.info/closure
created: 2026-07-09
tags:
  - javascript
---
## Code blocks

If a variable is declared inside a code block `{...}`. it's only visible inside that block.

```js
{
  // do some job with local variables that should not be seen outside

  let message = "Hello"; // only visible in this block

  alert(message); // Hello
}

alert(message); // Error: message is not defined
```

For `if`, `for`, `while` and so on, variables declared in `{...}` are only visible inside the code block.

## Nested functions

A function is called "nested" when it is created inside another function.

```js
function sayHiBye(firstName, lastName) {

  // helper nested function to use below
  function getFullName() {
    return firstName + " " + lastName;
  }

  alert( "Hello, " + getFullName() );
  alert( "Bye, " + getFullName() );

}
```

Here the _nested_ function `getFullName()` is made for convenience. It can access the outer variables and so can return the full name.

A nested function can be returned: either as a property of a new object or as a result by itself. It can then be used somewhere else. **No matter where, it still has access to the same outer variables.**

But how does it work, let's learn it step by step.

## Lexical Environment

For clarity, the explanation is split into multiple steps.

### Step 1. Variables

In JS, every running function, code block `{...}`, and the script as a whole have an internal (hidden) object known as the _Lexical environment_.

This lexical environment consists of two parts:
1. _Environment record_ - an object that stores all local variables as its properties (and some other information like the value of `this`).
2. A reference to the outer _lexical environment_, the one associated with the outer code.

**A “variable” is just a property of the special internal object, `Environment Record`. “To get or change a variable” means “to get or change a property of that object”.**

It looks something like this:

![](../../assets/Pasted%20image%2020260709100332.png)

This is the so-called _global_ lexical environment, associated with the whole script.

In the picture above, the rectangle means _environment record_ (variable store) and the arrow means outer reference. The global lexical environment has no outer reference, that's why the arrow points to `null`.

As the code starts executing this environment changes.

![](../../assets/Pasted%20image%2020260709100547.png)

Rectangle (on RHS) shows how the lexical environment changes during the execution.
1. When the script starts, the lexical environment is pre-populated with all declared variables. Initially, they are in "uninitialized" state, it's a special state in which the JS engine knows about the variable but you cannot use it until it has been declared with `let`.
2. Then `let phrase` defines the variables. There's no assignment yet, so the value is `undefined`. The variable is now available for use now.
3. `phrase` is assigned a value.
4. `phrase` changes the value.

- A variable is a property of a special internal object, associated with the currently executing block/function/script.
- Working with variables is actually working with the properties of that object.

### Lexical environment is a specification object
"Lexical" environment is a specification object. It only exists theoretically. We can't get this object in our code and manipulate it directly. 

### Step 2. Function declarations

A function is also a value, like a variable. **The difference is that a function declaration is instantly fully initialized.**

When a lexical environment is created, a function declaration immediately becomes a read-to-use function (unlike `let`, that is unusable till the declaration).

![](../../assets/Pasted%20image%2020260709101801.png)

Naturally, this behaviour only applies to Function declarations, not function expressions where we assign a function to a variable, such as `let say = function(name) ...`.

### Step 3. Inner and outer lexical environment

When a function runs, at the beginning of the call, a new lexical environment is created automatically to store local variables and parameters of the call.

![](../../assets/Pasted%20image%2020260709102140.png)

During the function call, we have two lexical environment, the inner one (for the function call) and the outer one (global).
- The inner lexical env corresponds to the _current execution_ of `say`. It has a single property: `name`, the function argument. We called `say('John')`, so the value of the `name` is `John`.
- The outer lexical env is the global lexical env. It has the `phrase` variable and the function itself. Inner lexical env references this outer one.

**When the code wants to access a variable – the inner Lexical Environment is searched first, then the outer one, then the more outer one and so on until the global one.**

If a variable is not found, it will generate a error in strict mode.

### Step 4. Returning a function

```js
function makeCounter() {
  let count = 0;

  return function() {
    return count++;
  };
}

let counter = makeCounter();
```

At the beginning of each `makeCounter()` call, a new lexical env object is created.

So we have two nested lexical env, just like in the example above:

![](../../assets/Pasted%20image%2020260709102805.png)

What's different is that, during the execution of `makeCounter`, a tiny nested function is created for the line `return count++`. We don't run it yet, only create. 

All functions remember the Lexical Environment in which they were made. All functions have the hidden property named `[[Environment]]`, that keeps the reference to the Lexical Environment where the function was created:

![](../../assets/Pasted%20image%2020260709103059.png)

So `counter.[[Enviornment]]` has the reference to `{count: 0}`. That's how the function remembers where it was created, no matter where it's called. The `[[Enviornment]]` reference is set once and forever at function creation time. 

Later, when `counter()` is called, a new lexical env is created for the call, and its outer lexical env is taken from the `counter.[[Enviornment]]`:

![](../../assets/Pasted%20image%2020260709104245.png)

_Note: When the function was first made (execution of the outer function) it has a reference to `[[Enviornment]]` and than when it is called it, a new lexical env is created for the call, and its outer lexical env is taken from `[[Enviornment]]`._

**A variable is updated in the lexical env where it lives**

![](../../assets/Pasted%20image%2020260709104524.png)

#### Closure
A [closure](https://en.wikipedia.org/wiki/Closure_\(computer_programming\)) is a function that remembers its outer variables and can access them. It means that functions automatically remembers where they were created using a hidden `[[Enviornment]]` property, and then their code can access outer variables.

## Garbage collection

A lexical env is removed from the memory with all the variables after the function call (because there's no reference to it anymore).

However, if there's a nested function that is still reachable at the end of the function, then it has `[[Environment]]` property that references the lexical environment. It remains in the memory even after the call has finished.

```js
function f() {
  let value = 123;

  return function() {
    alert(value);
  }
}

let g = f(); // g.[[Environment]] stores a reference to the Lexical Environment
// of the corresponding f() call
```

If we call `f()` many times and save the result, all the corresponding lexical env are also kept in memory.

It will free the memory when there's no reference to it.

```js
g = null;
```

## Real-life optimizations

JS engines try to optimize the variables that are kept in memory. They analyze variable usage and if it’s obvious from the code that an outer variable is not used – it is removed.

**An important side effect in V8 (Chrome, Edge, Opera) is that such variable will become unavailable in debugging.**

```js
function f() {
  let value = Math.random();

  function g() {
    debugger; // in console: type alert(value); No such variable!
  }

  return g;
}

let g = f();
g();
```

If you run the above code in developers tool, `value` would have been removed. 

Another such example:

```js
let value = "Surprise!";

function f() {
  let value = "the closest value";

  function g() {
    debugger; // in console: type alert(value); Surprise!
  }

  return g;
}

let g = f();
g();
```

