---
title: Removing Effect Dependencies
source: https://react.dev/learn/removing-effect-dependencies
created: 2026-09-17
tags:
  - react
---
## Dependencies should match the code

When you write an Effect, you first specify how to start and stop whatever you want your Effect to be doing, and add a cleanup function if needed.

Every reactive value used inside your Effect should be present in the dependency array.

## To remove a dependency, prove that it's not needed

One way to do this is to prove that a value is not reactive by moving it outside your component. Now your component won't re-render due to a change in that value, and the linter won't flag it as a missing dependency.

## Removing unnecessary dependencies

> If something happens due to a user interaction, that logic should belong in an event handler.

### Are you reading some state to calculate the next state?

The Effect below updates the `messages` state variable with a newly created array every time a new message arrives:

```jsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages([...messages, receivedMessage]);
    });
    // ...
```

Since `messages` is a reactive value, you'd need to add it to the dependency array — but that causes a problem.

Every time you receive a message, `setMessages()` causes the component to re-render with a new `messages` array that includes the received message. But since this Effect now depends on `messages`, this will _also_ re-synchronize the Effect. So every new message would make the chat re-connect. The user wouldn't like that!

To fix this, don't read `messages` inside the Effect. Instead, pass an updater function to `setMessages`:

```jsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages(msgs => [...msgs, receivedMessage]);
    });
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
  // ...
```

### Do you want to read a value without "reacting" to its changes?

Use Effect Events. _Read the docs._

#### Wrapping an event handler from the props

Say the component receives an event handler as a prop:

```jsx
function ChatRoom({ roomId, onReceiveMessage }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      onReceiveMessage(receivedMessage);
    });
    return () => connection.disconnect();
  }, [roomId, onReceiveMessage]); // ✅ All dependencies declared
  // ...
```

And the event handler is a new function on every render:

```jsx
<ChatRoom
  roomId={roomId}
  onReceiveMessage={receivedMessage => {
    // ...
  }}
/>
```

Since `onReceiveMessage` is a dependency, it would cause the Effect to re-synchronize after every parent re-render — reconnecting to the chat each time. To fix this, wrap the call in an Effect Event:

```jsx
function ChatRoom({ roomId, onReceiveMessage }) {
  const [messages, setMessages] = useState([]);

  const onMessage = useEffectEvent(receivedMessage => {
    onReceiveMessage(receivedMessage);
  });

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      onMessage(receivedMessage);
    });
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
  // ...
```

#### Separating reactive values and non-reactive code

Say you want to log a visit every time `roomId` changes, and include the current `notificationCount` with each log — but a change to `notificationCount` alone shouldn't trigger a log.

Split the non-reactive part into an Effect Event:

```jsx
function Chat({ roomId, notificationCount }) {
  const onVisit = useEffectEvent(visitedRoomId => {
    logVisit(visitedRoomId, notificationCount);
  });

  useEffect(() => {
    onVisit(roomId);
  }, [roomId]); // ✅ All dependencies declared
  // ...
}
```

You want your logic to react to `roomId`, so you read `roomId` inside the Effect. But you don't want a change to `notificationCount` to log an extra visit, so you read `notificationCount` inside the Effect Event instead.

### Does some reactive value change unintentionally?

Sometimes you do want your Effect to "react" to a value, but that value changes more often than it should.

```jsx
function ChatRoom({ roomId }) {
  // ...
  const options = {
    serverUrl: serverUrl,
    roomId: roomId
  };

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    // ...
```

Every time this component re-renders (due to unrelated state), the Effect re-runs because `options` is a new object each time. In JS, objects and functions are considered different even when their contents are the same.

To prevent this:

#### Move static objects and functions outside your component

If the object doesn't depend on any props or state, move it outside the component:

```jsx
const options = {
  serverUrl: 'https://localhost:1234',
  roomId: 'music'
};

function ChatRoom() {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, []); // ✅ All dependencies declared
  // ...
```

This works for functions too:

```jsx
function createOptions() {
  return {
    serverUrl: 'https://localhost:1234',
    roomId: 'music'
  };
}

function ChatRoom() {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const options = createOptions();
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, []); // ✅ All dependencies declared
  // ...
```

#### Move dynamic objects and functions inside your Effect

If the object depends on a reactive value (like a `roomId` prop) that can change on re-render, you can't move it outside the component — but you can move its creation inside the Effect:

```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
  // ...
```

Now `options` is declared inside the Effect, so it's no longer a dependency. The only reactive value the Effect depends on is `roomId` — a primitive, so it can't change unintentionally the way an object can.

#### Read primitive values from objects

Sometimes you receive an object from props:

```jsx
function ChatRoom({ options }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // ✅ All dependencies declared
  // ...
```

The risk is that the parent creates this object fresh on every render:

```jsx
<ChatRoom
  roomId={roomId}
  options={{
    serverUrl: serverUrl,
    roomId: roomId
  }}
/>
```

This causes the Effect to re-connect every time the parent re-renders. Fix it by reading primitive values from the object _outside_ the Effect, so you avoid object/function dependencies:

```jsx
function ChatRoom({ options }) {
  const [message, setMessage] = useState('');

  const { roomId, serverUrl } = options;
  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // ✅ All dependencies declared
  // ...
```

Now, if the parent recreates `options` unintentionally, the chat won't re-connect. But if `options.roomId` or `options.serverUrl` actually change, it will.

## Recap

- Dependencies should always match the code.
- When you're not happy with your dependencies, edit the code, not the dependency array.
- Suppressing the linter leads to confusing bugs — always avoid it.
- To remove a dependency, you need to "prove" to the linter that it's unnecessary.
- If code should run in response to a specific interaction, move it to an event handler.
- If different parts of your Effect should re-run for different reasons, split it into several Effects.
- If you want to update state based on the previous state, pass an updater function.
- If you want to read the latest value without "reacting" to it, extract an Effect Event from your Effect.
- In JavaScript, objects and functions are considered different if they were created at different times.
- Avoid object and function dependencies where possible — move them outside the component or inside the Effect.
