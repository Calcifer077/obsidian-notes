---
title: Reacting to Input with State
source: https://react.dev/learn/reacting-to-input-with-state
created: 2026-08-12
tags:
  - react
---
## Declarative vs. Imperative UI

**Imperative** programming focuses on *how* to do something. You have to give your framework exact, step‑by‑step instructions to manipulate the UI based on what just happened. For example, if you're building a form, you have to explicitly tell it to enable, disable, show, or hide various components for every possible scenario. It's like riding next to someone in a car and giving them turn‑by‑turn directions. One wrong instruction, and you end up in the wrong place.

**Declarative** programming focuses on *what* to do. You declare what you want to show for each state, and React figures out *how* to update the UI. It's like telling a taxi driver your destination instead of dictating every turn. The driver handles the route, and they might even know a shortcut you hadn't considered.

The declarative approach shines in complex systems. While imperative code works well for isolated examples, it becomes exponentially harder to manage as the system grows—adding a single new UI element or interaction requires carefully reviewing all existing code to ensure you haven't introduced a bug (e.g., forgetting to show or hide something).

## The Five Steps to Think About UI Declaratively

### Step 1: Identify Your Component's Different Visual States

Before writing any logic, visualize all the possible states a user might see. Take a form as an example:

- **Empty**: The "Submit" button is disabled.
- **Typing**: The "Submit" button is enabled.
- **Submitting**: The form is completely disabled and a loading spinner is shown.
- **Success**: A thank‑you message is displayed and the form is hidden.
- **Error**: Same as the typing state, but with an error message displayed.

### Step 2: Determine What Triggers Those State Changes

Two types of inputs trigger state updates:

- **Human inputs**: Clicking a button, typing in a field, navigating via links, etc.
- **Computer inputs**: Network responses arriving, timers completing, images loading, etc.

**Pro tip**: Draw each state as a labeled circle and every transition as an arrow. This helps you spot potential issues before you even start coding.

### Step 3: Represent the State in Memory Using `useState`

Start with the state that absolutely must exist. For the form example:

```javascript
const [answer, setAnswer] = useState('');
const [error, setError] = useState(null);
```

It's okay to start with boolean state variables if you're unsure; you can refine them later.

### Step 4: Remove Any Non-Essential State Variables

The goal is to prevent having state in memory that doesn't represent a valid UI. Ask yourself these questions:

- **Does this state cause contradictions?** For instance, `isTyping` and `isSubmitting` should never be `true` at the same time. Consider merging them into a single `status` state variable (e.g., `'typing'`, `'submitting'`, `'success'`).
- **Is this information already available elsewhere?** For example, you can remove `isEmpty` and simply check `answer.length === 0`.
- **Can you derive it from the inverse of another state?** For instance, you can remove `isError` and simply check `error !== null`.

By pruning effectively, you can often reduce seven booleans down to three core variables:

```javascript
const [answer, setAnswer] = useState('');
const [error, setError] = useState(null);
const [status, setStatus] = useState('typing'); // 'typing' | 'submitting' | 'success'
```

### Step 5: Connect the Event Handlers to Set the State

Wire up your event handlers to update the state accordingly. For example:

- On text input change: toggle between the "Empty" and "Typing" states based on whether the input field is empty.
- On clicking the submit button: switch to the "Submitting" state.
- After a network request succeeds: switch to the "Success" state.
- After a network request fails: switch to the "Error" state and display the error message.

## Recap 

React offers a declarative way to handle the UI. Instead of directly manipulating individual UI parts, you describe the different states your component can be in and switch between them based on user input. 

## Further reading 

- [fireship.dev - Imperative VS Declarative programming](https://fireship.dev/c/react/imperative-vs-declarative)