---
title: Scaling up with reducer and context
source: https://react.dev/learn/scaling-up-with-reducer-and-context
created: 2026-08-17
tags:
  - react
---
Reducers let you consolidate a component's state update logic. Context lets you pass information deep down to other components. You can combine both to manage state of a complex screen.

Here is how you can combine a reducer with context:

1. Create the context.
2. Put state and dispatch into context.
3. Use context anywhere in the tree.

We will use the example from [Extracting state logic into a Reducer](Extracting%20state%20logic%20into%20a%20Reducer.md).

### Step 1. Create the context 

The `useReducer` hook returns the current `tasks` and the `dispatch` function that lets you update them:

```JSX
const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
```

To pass them down the tree, you will create two separate contexts:

- `TasksContext` provides the current list of tasks.
- `TasksDispatchContext` provides the function that lets components dispatch actions.

Export them from a separate file so that you can later import them from other files:

```JSX
// TasksContext.js
import { createContext } from 'react';

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);
```

### Step 2. Put state and dispatch into context 

Now you can import both contexts in your component. 

```JSX
import { TasksContext, TasksDispatchContext } from './TasksContext.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
  // ...
  return (
    <TasksContext value={tasks}>
      <TasksDispatchContext value={dispatch}>
        ...
      </TasksDispatchContext>
    </TasksContext>
  );
}
```

### Step 3. Use context anywhere in the tree 

Now you don't need to pass the list of tasks or the event handlers down the tree, instead any component that needs the task list (state) can read it from the `TasksContext`:

```JSX
export default function TaskList() {
  const tasks = useContext(TasksContext);
  // ...
```

To update the task list, any component can read the `dispatch` function from context and call it:

```JSX
export default function AddTask() {
  const [text, setText] = useState('');
  const dispatch = useContext(TasksDispatchContext);
  // ...
  return (
    // ...
    <button onClick={() => {
      setText('');
      dispatch({
        type: 'added',
        id: nextId++,
        text: text,
      });
    }}>Add</button>
    // ...
```

You could also move both reducer and context into a single file.

## Recap

- You can combine reducer with context to let any component read and update state above it.
- To provide state and the dispatch function to components below:
    1. Create two contexts (for state and for dispatch functions).
    2. Provide both contexts from the component that uses the reducer.
    3. Use either context from components that need to read them.
- You can further declutter the components by moving all wiring into one file.
    - You can export a component like `TasksProvider` that provides context.
    - You can also export custom Hooks like `useTasks` and `useTasksDispatch` to read it.
- You can have many context-reducer pairs like this in your app.