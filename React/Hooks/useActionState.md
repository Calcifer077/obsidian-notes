`useActionState` is a React hook that lets you update your component's state when you do something that takes time (like sending data to a server, saving something, or calling an API).

Normally, when you click a button or submit a form, you might want to:
- Show a `loading` message or spinner
- Update some number or list on the screen
- Wait for the server to reply

`useActionState` makes this easy and safe. It automatically tells you when the action is still happening (so you can show loading) and updates the state nicely.

**`Actions`** are a new primitive for managing asynchronous operations (like form submissions or data mutations).
- An action is an asynchronous function that uses Transitions to automatically manage the state of a data mutation
- They automatically track pending states, handle errors, and support optimistic updates.

Usage of `useActionState`:
```jsx
const [state, dispatchAction, isPending] = useActionState(reducerAction, initialState, permalink?);
```

Call `useActionState` at the top level of your component to create state for the result of an action.

```jsx
import { useActionState } from 'react';  

function reducerAction(previousState, actionPayload) {  
// ...  
}  

function MyCart({initialState}) {  
const [state, dispatchAction, isPending] = useActionState(reducerAction, initialState);  
// ...  
}
```
#### Parameters:
##### 1. `reducerAction`
The function to be called when the Action is triggered. When called, it receives the previous state (initially the `initialState` you provided, then its previous return value) as its first argument, followed by the `actionPayload` passed to `dispatchAction`. 
It can do slow things (like talking to a server) and must return the new state. When it returns the new state, it triggers a Transition to re-render with that state.
Unlike reducers in `useReducer`, the `reducerAction` can be async and perform side effects. Each time you call `dispatchAction`, React calls the `reducerAction` with the `actionPayload`, `userData` in this case.
```js
async function addToCartAction(previousCount, userData) {
  // You can wait for the server here
  await someApiCall();
  return previousCount + 1;   // new state
}
```

##### 2. `initialState`
The value you want the state to be initially. React ignores this argument after `dispatchAction` is invoked for the first time meaning that react only uses this once.

##### 3. permalink (optional)
This is only needed if you're using React on the server with forms. It tells the browser where to go if JavaScript hasn't loaded yet.
#### Returns:
`useActionState` returns an array with exactly three values:
1. The current state. During the first render, it will match the `initialState` you passed. After `dispatchFunction` is invoked, it will match the value returned by the `reducerAction`.
2. A `dispatchAction` function you call when the user clicks a button or submits a form.
3. The `isPending` flag that tells you if any dispatched Actions for this Hook are pending. `true` while action is running, `false` when it's finished.

A very simple example:
```jsx
import { useActionState } from 'react';

async function addToCartAction(previousCount, userData) {
  // You can wait for the server here
  await someApiCall();
  return previousCount + 1;   // new state
}

function MyCounter() {
  const [count, dispatch, isPending] = useActionState(addToCartAction, 0);

  return (
    <div>
      <p>Tickets in cart: {count}</p>
      
      <button 
        onClick={() => dispatch()}   // ← this runs your action
        disabled={isPending}        // ← disable button while loading
      >
        {isPending ? 'Adding...' : 'Add one ticket'}
      </button>
    </div>
  );
}

```
When the user clicks the button:
- `isPending` becomes `true`
- Your action function runs
- When it finishes, `count` updates to the new value
- `isPending` becomes `false`

	You must call `dispatchAction` when the user does something (button click, form submit). Don't call it while the component is drawing the screen.
	If something goes wrong in your action, React can show an error message.

#### Things to know
- `useActionState` is a Hook, so you can only call it **at the top level of your component** or your own Hooks. You can’t call it inside loops or conditions. If you need that, extract a new component and move the state into it.
- React queues and executes multiple calls to `dispatchAction` sequentially. Each call to `reducerAction` receives the result of the previous call. What it means is that if you say click a button repeatedly which does some work like incrementing some count. React will batch all your clicks and update count one by one. [Check out more](https://react.dev/reference/react/useActionState#adding-state-to-an-action)
- The `dispatchAction` function has a stable identity, so you will often see it omitted from Effect dependencies, but including it will not cause the Effect to fire. If the linter lets you omit a dependency without errors, it is safe to do.
- When using Server Functions, `initialState` and `actionPayload` needs to be [serializable](https://react.dev/reference/rsc/use-server#serializable-parameters-and-return-values) (values like plain objects, arrays, strings, and numbers).
- If `dispatchAction` throws an error, React cancels all queued actions and shows the nearest [Error Boundary](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary). You can also cancel queued actions using [`AbortController`](https://react.dev/reference/react/useActionState#cancelling-queued-actions).  
- If there are multiple ongoing Actions, React batches them together. This is a limitation that may be removed in a future release.
- `reducerAction` is not invoked twice in `<StrictMode>` since `reducerAction` is designed to allow side effects.
- The return type of `reducerAction` must match the type of `initialState`.

Link to docs:
[Docs](https://react.dev/reference/react/useActionState)
