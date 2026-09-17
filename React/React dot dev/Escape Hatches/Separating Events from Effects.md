---
title: Separating Events from Effects
source: https://react.dev/learn/separating-events-from-effects
created: 2026-09-17
tags:
  - react
---
## Choosing between event handlers and Effects 

Imagine you're implementing a chat room component. Your requirements look like this:

1. Your component should automatically connect to the selected chat room.
2. When you click the "Send" button, it should send a message to the chat.

Let's say we already have their implementations but are not sure where to put them.

### Event handlers run in response to specific interactions

From the user's perspective, sending a message should happen because a "Send" button was clicked. This is why sending a message should be a event handler.

### Effects run whenever synchronization is needed 

Recall that we also need to connect to a chat room. Where does this code go?

The reason to run this code is not because user clicked something, it's just because user navigated to the chat room screen and now we need to connect to the chat room. Effect ensures that the component will remain synchronized with the currently selected room.

## Reactive values and reactive logic 

One can say that event handlers are always triggered "manually", for example by clicking a button. Effects, on the other hand, are "automatic": they run and re-run as often as it's needed to stay synchronized. 

Reactive values like props, state and variables declared inside your component can change due to a re-render. Event handlers and Effects response to these changes differently:

- **Logic inside event handlers is not reactive**. It will not run again unless the user performs the same interactive again. Event handlers can read reactive values without "reacting" to their changes.
- **Logic inside Effects is reactive**. If your Effect reads a reactive value, you have to specify it as a dependency. Then, if a re-render causes that value to change, React will re-run your Effect's logic with the new value. 

## Extracting non-reactive logic out of Effects 

Imagine that your want to show a notification when the user connects to the chat. You read the current theme (dark or light) from the props so that you can show the notification in the correct color:

```JSX
function ChatRoom({ roomId, theme }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      showNotification('Connected!', theme);
    });
    connection.connect();
    // ...
```

However, `theme` is a reactive value and you need to mention it in the dependency array.

```JSX
[roomId, theme]
```

But it would cause a problem. Every time user changes the `theme` effect will re-run showing the notification even though `roomId` remained the same. 

You need a way to separate this non-reactive logic from the reactive Effect around it. 

## Declaring an Effect Event 

Use a special hook called `useEffectEvent` to extract this non-reactive logic out of your Effect:

```JSX
import { useEffect, useEffectEvent } from 'react';

function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => {
    showNotification('Connected!', theme);
  });
  // ...
```

Here, `onConnected` is called an Effect Event. It's a part of your logic, but it behaves a lot more like an event handler. The logic inside it is not reactive, and it always "sees" the latest values of your props and state.

Now you can call `onConnected` Effect Event from inside your Effect:

```JSX
function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => {
    showNotification('Connected!', theme);
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      onConnected();
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
  // ...
```

You don't need to mention `onConnected` to the Effect's dependencies because Effect Events are not reactive and must be omitted from dependencies.

You can think of Effect Events as being very similar to event handlers. The main difference is that event handlers run in response to user interactions, whereas Effect Events are triggered by you from Effects.

## Limitations of Effect Events 

- Only call them from inside effects
- Never pass them to other components or Hooks.

Effect Events are non-reactive “pieces” of your Effect code. They should be next to the Effect using them.

## Recap 

- Event handlers run in response to specific interactions.
- Effects run whenever synchronization is needed.
- Logic inside event handlers is not reactive.
- Logic inside Effects is reactive.
- You can move non-reactive logic from Effects into Effect Events.
- Only call Effect Events from inside Effects.
- Don’t pass Effect Events to other components or Hooks.