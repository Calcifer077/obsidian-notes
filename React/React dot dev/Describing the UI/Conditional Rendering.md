---
title: Conditional Rendering
source: https://react.dev/learn/conditional-rendering
created: 2026-07-29
tags:
  - react
---
Your components will often need to display different things depending on different conditions. In React, you can conditionally render JSX using JavaScript syntax like `if` statements, `&&`, and `? :` operators.

## Conditionally returning JSX

Let's say you want to render some items conditionally based on some condition.

One way to do so is [`if`/`else` statement](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/if...else) like so:

```JSX
if (isPacked) {
  return <li className="item">{name} ✅</li>;
}
return <li className="item">{name}</li>;
```

If the `isPacked` prop is `true`, this code returns a different JSX tree. 

### Conditionally returning nothing with `null`

In some situations, you won't want to render anything at all but a component must return something. In this case, you can return `null`:

```JSX
if (isPacked) {
  return null;
}
return <li className="item">{name}</li>;
```

If `isPacked` is true, the component will return nothing, `null`. Otherwise, it will return JSX to render.

## Conditionally including JSX

### Conditional (ternary) operator (`? :`)

JavaScript has a compact syntax for writing a conditional expression — the [conditional operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_Operator) or “ternary operator”.

Instead of this:

```JSX
if (isPacked) {
  return <li className="item">{name} ✅</li>;
}
return <li className="item">{name}</li>;
```

You can write this:

```JSX
return (
  <li className="item">
    {isPacked ? name + ' ✅' : name}
  </li>
);
```

You can read it as _“if `isPacked` is true, then (`?`) render `name + ' ✅'`, otherwise (`:`) render `name`”_.

### Logical AND operator (`&&`)

Another common shortcut you’ll encounter is the [JavaScript logical AND (`&&`) operator.](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_AND#:~:text=The%20logical%20AND%20\(%20%26%26%20\)%20operator,it%20returns%20a%20Boolean%20value.) Inside React components, it often comes up when you want to render some JSX when the condition is true, **or render nothing otherwise.** With `&&`, you could conditionally render the checkmark only if `isPacked` is `true`:

```JSX
return (
  <li className="item">
    {name} {isPacked && '✅'}
  </li>
);
```

You can read this as _“if `isPacked`, then (`&&`) render the checkmark, otherwise, render nothing”_.

A [JavaScript && expression](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_AND) returns the value of its right side (in our case, the checkmark) if the left side (our condition) is `true`. But if the condition is `false`, the whole expression becomes `false`. React considers `false` as a “hole” in the JSX tree, just like `null` or `undefined`, and doesn’t render anything in its place.

## Recap

- In React, you control branching logic with JavaScript.
- You can return a JSX expression conditionally with an `if` statement.
- You can conditionally save some JSX to a variable and then include it inside other JSX by using the curly braces.
- In JSX, `{cond ? <A /> : <B />}` means _“if `cond`, render `<A />`, otherwise `<B />`”_.
- In JSX, `{cond && <A />}` means _“if `cond`, render `<A />`, otherwise nothing”_.
- The shortcuts are common, but you don’t have to use them if you prefer plain `if`.