---
title: Passing Props to a Component
source: https://react.dev/learn/passing-props-to-a-component
created: 2026-07-29
tags:
  - react
---
React components use _props_ to communicate with each other. Every parent component can pass some information to its child components by giving them props. Props can contain any JS value including objects, arrays, and functions.

## Familiar props

Props are the information that you pass to a JSX tag. For example, `className`, `src`, `alt`, `width`, and `height` are some of the props you can pass to an `<img>`:

```JSX
function Avatar() {
  return (
    <img
      className="avatar"
      src="https://react.dev/images/docs/scientists/1bX5QH6.jpg"
      alt="Lin Lanying"
      width={100}
      height={100}
    />
  );
}

export default function Profile() {
  return (
    <Avatar />
  );
}
```

## Passing props to a component

In above code, the `Profile` component isn't passing any props to its child component, `Avatar`:

```JSX
export default function Profile() {
  return (
    <Avatar />
  );
}
```

You can give `Avatar` some props in two steps.

### Step 1: Pass props to the child component

First, pass some props to `Avatar`. For example, let’s pass two props: `person` (an object), and `size` (a number):

```JSX
export default function Profile() {
  return (
    <Avatar
      person={{ name: 'Lin Lanying', imageId: '1bX5QH6' }}
      size={100}
    />
  );
}
```

### Step 2: Read props inside the child component

You can read these props by listing their names `person`, `size` separated by the commas inside `({` and `})` inside function declaration.

```JSX
function Avatar({ person, size }) {
  // person and size are available here
}
```

props _are_ the only argument to your component. React component functions accept a single argument, a `props` object:

```JSX
function Avatar(props) {
  let person = props.person;
  let size = props.size;
  // ...
}
```

## Specifying a default value for a prop

If you want to give a prop a default value to fall back on when no value is specified, you can do it with the destructuring by putting `=` and the default value right after the parameter.

```JSX
function Avatar({ person, size = 100 }) {
  // ...
}
```

The default value is only used if the `size` prop is missing or if you pass `size={undefined}`. But if you pass `size={null}` or `size={0}`, the default value will **not** be used.

## Forwarding props with the JSX spread syntax

If all a component do with received props is just to pass it to its child component we can do so using spread operator.

```JSX
function Profile(props) {
  return (
    <div className="card">
      <Avatar {...props} />
    </div>
  );
}
```

## Passing JSX as children

You can next your components into one another. When you nest content inside a JSX tag, the parent component will receive that content in a prop called `children`.

```JSX
// Avatar.js
import { getImageUrl } from './utils.js';

export default function Avatar({ person, size }) {
  return (
    <img
      className="avatar"
      src={getImageUrl(person)}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}
```

```JSX
import Avatar from './Avatar.js';

function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}

export default function Profile() {
  return (
    <Card>
      <Avatar
        size={100}
        person={{
          name: 'Katsuko Saruhashi',
          imageId: 'YfeOqp2'
        }}
      />
    </Card>
  );
}
```

You can think of a component with a `children` prop as having a "hole" that can be "filled in" by its parent components with arbitrary JSX.

![](../../../assets/Pasted%20image%2020260729175742.png)

## Recap

- To pass props, add them to the JSX, just like you would with HTML attributes.
- To read props, use the `function Avatar({ person, size })` destructuring syntax.
- You can specify a default value like `size = 100`, which is used for missing and `undefined` props.
- You can forward all props with `<Avatar {...props} />` JSX spread syntax, but don’t overuse it!
- Nested JSX like `<Card><Avatar /></Card>` will appear as `Card` component’s `children` prop.
- Props are read-only snapshots in time: every render receives a new version of props.
- You can’t change props. When you need interactivity, you’ll need to set state.