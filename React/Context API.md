# React Context API: Passing Data Deeply

The React Context API provides a way to share values like themes, user authentication status, or preferred language between components **without** having to explicitly pass props through every level of the component tree. This technique is designed to prevent what is known as **prop drilling**.

The core idea is that a parent component uses a **Provider** to define a value, and any component deep in the tree below it can use a **Consumer** or the `useContext` hook to access that value directly.

---

## Key Principles of Context

The behavior of Context can be likened to how cascading style sheets (CSS) properties work, specifically regarding their scope and inheritance.

### 1\. Scope (Inheritance)

Context values are available to **all components below their Provider**, regardless of how many intermediate components exist. The context value will "cascade" down the component tree until it hits a new Provider for the same context.

- **Analogy:** If you set `font-family: Arial;` on a `<div>`, all elements inside that `<div>` will inherit that font unless a more specific rule changes it. The context value is inherited by all children.

### 2\. Overriding (The Closest Provider Wins)

If you declare the same Context multiple times within the component tree, the value from the **closest Provider (going upwards from the consumer)** will be used. A child Provider effectively overrides the value of its ancestor Provider for its entire subtree.

- **Analogy:** If a parent `<div>` has `color: blue;` and a child `<p>` inside it has `color: red;`, the text in the `<p>` will be red because the `<p>`'s CSS rule is closer and more specific. The same principle applies to Context.

---

## Example: Demonstrating Inheritance and Overriding

This example uses a `ThemeContext` to demonstrate how a value is inherited and then selectively overridden in a subtree.

### 1\. Creating the Context (`ThemeContext.js`)

We create the context with a default value of `'default'`. This value is used only when a component consumes the context _outside_ of any Provider.

```javascript
// ThemeContext.js
import { createContext } from "react";

// createContext('default') sets the value when no Provider is found
export const ThemeContext = createContext("default");
```

Another pattern could be that we can provide context as a hook:
```jsx
function useThemes() {
  const context = useContext(ThemeContext);

  if (!context) {
    throw new Error("useThemes must be used within ThemeContext");
  }

  return context;
}
```
You can export the above function and anyone below it can access it using `const {theme} = useThemes();` instead of `const theme = useContext(ThemeContext)`
### 2\. The Consumer Component (`ThemedButton.js`)

This component uses the `useContext` hook to access the theme value from the nearest Provider above it.

```javascript
// ThemedButton.js
import React, { useContext } from "react";
import { ThemeContext } from "./ThemeContext";

function ThemedButton({ label }) {
  // Finds the value from the nearest ThemeContext.Provider above it
  const theme = useContext(ThemeContext);

  const style = {
    padding: "10px",
    color: theme === "dark" ? "white" : "black",
    backgroundColor: theme,
    border: "2px solid black",
    margin: "5px",
    borderRadius: "4px",
    cursor: "pointer",
  };

  return (
    <button style={style}>
      {label} (Theme: {theme})
    </button>
  );
}
export default ThemedButton;
```

### 3\. The Application Structure (`App.js`)

The main component sets up an **outer Provider** and then places an **inner Provider** to demonstrate the overriding principle.

```javascript
// App.js
import React from "react";
import { ThemeContext } from "./ThemeContext";
import ThemedButton from "./ThemedButton";

function IntermediateContainer() {
  return (
    <div style={{ margin: "10px", padding: "10px", border: "1px dashed gray" }}>
      <h4>Intermediate Container (Inherits from parent)</h4>
      <ThemedButton label="Button in Middle Component" />
    </div>
  );
}

function App() {
  return (
    <div>
      <h1>Context Overriding in Action</h1>

      {/* 1. OUTER PROVIDER: Sets context value to 'dark' for its entire subtree */}
      <ThemeContext.Provider value="dark">
        <h2>Section A: Outer 'dark' Theme</h2>
        <ThemedButton label="Button A1" />

        {/* Component B inherits 'dark' from the outer provider */}
        <IntermediateContainer />

        {/* 2. INNER PROVIDER: Overrides context value to 'blue' for its subtree only */}
        <ThemeContext.Provider value="blue">
          <h3>Section B: Inner 'blue' Theme (Override)</h3>
          <ThemedButton label="Button B2" />
        </ThemeContext.Provider>

        {/* Context reverts to the nearest Provider above it (the outer 'dark' one) */}
        <h2>Section C: Reverting to Outer 'dark' Theme</h2>
        <ThemedButton label="Button C3" />
      </ThemeContext.Provider>

      {/* 3. NO PROVIDER: Component uses the Context's default value */}
      <h2>Section D: Outside All Providers</h2>
      <ThemedButton label="Button D4 (Default)" />
    </div>
  );
}
export default App;
```

### Demonstration Summary

| Component                      | Nearest Provider Value | Resulting Theme | Principle Demonstrated                            |
| :----------------------------- | :--------------------- | :-------------- | :------------------------------------------------ |
| **Button A1**                  | `dark`                 | `dark`          | **Inheritance** (from outer Provider)             |
| **Button in Middle Component** | `dark`                 | `dark`          | **Scope** (passes through intermediate component) |
| **Button B2**                  | `blue`                 | `blue`          | **Overriding** (inner Provider is closer)         |
| **Button C3**                  | `dark`                 | `dark`          | **Reversion** (inner Provider's scope ends)       |
| **Button D4**                  | None                   | `default`       | **Default Value** (outside all Providers)         |

Where I have used this? worldwise, lively votes (auth)