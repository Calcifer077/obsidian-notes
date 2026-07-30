---
title: Understanding Your UI as a Tree
source: https://react.dev/learn/understanding-your-ui-as-a-tree
created: 2026-07-30
tags:
  - react
---
React, and many other UI libraries, model UI as a tree. Thinking of your app as a tree is useful for understanding the relationship between components.

## Your UI as a tree

The UI is often represented using tree structures. For example, browsers use tree structures to model HTML ([DOM](https://developer.mozilla.org/docs/Web/API/Document_Object_Model/Introduction)) and CSS ([CSSOM](https://developer.mozilla.org/docs/Web/API/CSS_Object_Model)). Mobile platforms also use trees to represent their view hierarchy.

![](../../../assets/Pasted%20image%2020260730154544.png)

These trees are useful tools to understand how data flows through a React app and how to optimize rendering and app size.

## The Render tree

As we know that we can _nest components_. Due to which we have concept of parent and child components, where each parent component may itself be a child of another component.

When we render a React app, we can model this relationship in a tree, known as the render tree.

```JSX
import FancyText from './FancyText';
import InspirationGenerator from './InspirationGenerator';
import Copyright from './Copyright';

export default function App() {
  return (
    <>
      <FancyText title text="Get Inspired App" />
      <InspirationGenerator>
        <Copyright year={2004} />
      </InspirationGenerator>
    </>
  );
}
```


![](../../../assets/Pasted%20image%2020260730154839.png)

From the example app, we can construct the above render tree.

The tree is composed of nodes, each of which represents a component. `App`, `FancyText`, `Copyright`, to name a few, are all nodes in our tree.

The root node in a React render tree is the [root component](https://react.dev/learn/importing-and-exporting-components#the-root-component-file) of the app. In this case, the root component is `App` and it is the first component React renders. Each arrow in the tree points from a parent component to a child component.

_Note: Render tree is only composed of React components, not HTML tags._

A render tree represents a single render pass of a React application. With [conditional rendering](Conditional%20Rendering.md), a parent component may render different children depending on the data passed. 

```JSX
import FancyText from './FancyText';
import InspirationGenerator from './InspirationGenerator';
import Copyright from './Copyright';

export default function App() {
  return (
    <>
      <FancyText title text="Get Inspired App" />
      <InspirationGenerator>
        <Copyright year={2004} />
      </InspirationGenerator>
    </>
  );
}
```

![](../../../assets/Pasted%20image%2020260730155200.png)

_Note: let's just assume the InspirationGenerator renders conditionally. You can check out original code here: [CodeSandBox](https://codesandbox.io/p/sandbox/r498wd?file=%2Fsrc%2FInspirationGenerator.js%3A1%2C1-23%2C1)_

In this example, depending on what `inspiration.type` is, we may render `<FancyText>` or `<Color>`. The render tree may be different for each render pass.

## The Module Dependency Tree

Another relationship in a React app that can be modeled with a tree are an app’s module dependencies. As we [break up our components](https://react.dev/learn/importing-and-exporting-components#exporting-and-importing-a-component) and logic into separate files, we create [JS modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) where we may export components, functions, or constants.

Each node in a module dependency tree is a module and each branch represents an `import` statement in that module.

If we take the previous Inspirations app, we can build a module dependency tree, or dependency tree for short.

![](../../../assets/Pasted%20image%2020260730155451.png)

Dependency trees are useful to determine what modules are necessary to run your React app. When building a React app for production, there is typically a build step that will bundle all the necessary JavaScript to ship to the client. The tool responsible for this is called a [bundler](https://developer.mozilla.org/en-US/docs/Learn/Tools_and_testing/Understanding_client-side_tools/Overview#the_modern_tooling_ecosystem), and bundlers will use the dependency tree to determine what modules should be included.

## Recap

- Trees are a common way to represent the relationship between entities. They are often used to model UI.
- Render trees represent the nested relationship between React components across a single render.
- With conditional rendering, the render tree may change across different renders. With different prop values, components may render different children components.
- Render trees help identify what the top-level and leaf components are. Top-level components affect the rendering performance of all components beneath them and leaf components are often re-rendered frequently. Identifying them is useful for understanding and debugging rendering performance.
- Dependency trees represent the module dependencies in a React app.
- Dependency trees are used by build tools to bundle the necessary code to ship an app.
- Dependency trees are useful for debugging large bundle sizes that slow time to paint and expose opportunities for optimizing what code is bundled.