
`lazy` is used in react for ==**code-splitting** and **performance optimization**== by allowing you to defer the loading of a component's code until it is needed for the first time, rather than loading everything upfront in the initial bundle. This technique improves the initial load time and overall performance of large applications, leading to a better user experience.

`lazy` lets you defer loading component’s code until it is rendered for the first time.

```jsx
import { lazy } from react;

const SomeComponent = lazy(load);
```

What is `load` in the above file?
**`load`** -> It is basically a function that returns a component. In technical terms it returns a promise. React will not call this load until the first time you try to render the component. After React first calls `load` it will wait for it to resolve. Both returned promise and promise's resolved value will be cached so that react will not call it more than once. If the promise rejects, it will store the rejected value and the nearest react error boundary will catch it.

`lazy` returns a react component that you can render in your tree. While the code for `lazy` is still loading it will be suspended. So it is better to wrap it in a Suspense boundary.

**Usage:**

Usually, you import component:

```jsx
import MarkdownPreview from './MarkdownPreview.js';
```

To defer load this component code until its rendered for the first time use dynamic import.

```jsx
import { lazy } from 'react';  

const MarkdownPreview = lazy(() => import('./MarkdownPreview.js'));
```

Above we are using dynamic loading, it loads files asynchronously.

Now that the component loads on demand, you need to tell what needs to be displayed while it loads:

```jsx
<Suspense fallback={<Loading />}>  
	<h2>Preview</h2>  
	<MarkdownPreview />  
</Suspense>
```
 Declare lazy components outside your component.

**When will the caching work?**
A lazily loaded module is fetched only once, even if you `React.lazy()` it in multiple places.

Example:
`lazyComponent.js`
```jsx
import { lazy } from "react";

export const MyComponent = lazy(() => import("./MyComponent"));
```

`PageA.jsx`
```jsx
import { MyComponent } from "./lazyComponents";

function PageA() {
  return <MyComponent />;
}
```

`PageB.jsx`
```jsx
import { MyComponent } from "./lazyComponents";

function PageB() {
  return <MyComponent />;
}
```

The chunk is downloaded **once**, even if you:
* Navigate between pages.
* You render it multiple times.
* You import it in multiple files.
The cache lives in ES module system, bundler chunk loader(Webpack / vite etc.), browser.

It would reload again only if you:
 * hard refresh the page.
 * deploy a new build.
 * 'r in dev mode with HMR.

**Short notes:**
1. To load component asynchronously.
2. Will cache the result even if same file is loaded across many files.
3. Use `suspense` around it.





