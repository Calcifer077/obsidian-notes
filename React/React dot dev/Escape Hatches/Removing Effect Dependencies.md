---
title: Removing Effect Dependencies
source: https://react.dev/learn/removing-effect-dependencies
created: 2026-09-17
tags:
  - react
---
## Dependencies should match the code 

When you write an effect, you first specify how to start and stop whatever you want your Effect to be doing, add a cleanup function. 

Every reactive value inside your code should be present in the dependency array. 

## To remove a dependency, prove that it's not a dependency 

One way of doing so it to prove that it is not a reactive value by moving it outside of your component. Now, your component won't re-render due to change in that value and the linter won't show error for it. 
## Removing unnecessary dependencies 

>If something happens due to a user interaction, that logic should belong to a event handler.

### Are you reading some state to calculate the next state?

Below effect updates the `messages` state variable with a newly created array every time a new message arrives:

```JSX
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

As `messages` is a reactive value, you need to insert it into the dependency array, which causes a problem. 

Every time you receive a message, `setMessages()` causes the component to re-render with a new `messages` array the included the received message. However, since this Effect now depends on `messages`, this will _also_ re-synchronize the Effect. So every new message will make the chat re-connect. The user would not like that!

To fix the issue, don't read `messages` inside the Effect. Instead, pass an updater function to `setMessages`:

```JSX
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

Use Effect events. _Read docs plsssss._

#### Wrapping an event handler from the props 

Say the component receives a event handler as a prop:

```JSX
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

And the event handler is different on every render:

```JSX
<ChatRoom
  roomId={roomId}
  onReceiveMessage={receivedMessage => {
    // ...
  }}
/>
```

Since `onReceiveMessage` is a dependency, it would cause the Effect to re-synchronize after every parent re-render. This would make it re-connect to the chat. To solve this, wrap the call in an Effect Event:

```JSX
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

In below example, you want to log a visit every time `roomId` changes. You want to include the current `notificationCount` with every log, but you don't want a change to `notificationCount` to trigger a log event.

The solution is again to split out the non-reactive code into an Effect Event:

```JSX
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

You want your logic to be reactive with regards to `roomId`, so you read `roomId` inside of your Effect. However, you don’t want a change to `notificationCount` to log an extra visit, so you read `notificationCount` inside of the Effect Event.

### Does some reactive value change unintentionally?

Sometimes, you do want your Effect to "react" to a certain value, but that value changes too often.

```JSX
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

Every time the above component re-renders (due to some different state), the effect will re-run because `options` will be a different object. In JS, every object and function are different even if their content is similar.

To prevent this, you can do the following:

#### Move static objects and functions outside your component 

If the object does not depend on any props and state, you can move that object outside your component:

```JSX
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

```JSX
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

If your object depends on some reactive value that may change as a result of a re-render, like a `roomId` prop, you can't pull it outside your component. You can, however, move its creation inside of your Effect's code:

```JSX
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

Now that `options` is declared inside of your Effect, it is no longer a dependency of your Effect. Instead, the only reactive value used by your Effect is `roomId`. Since `roomId` is not an object or function, you can be sure that it won’t be _unintentionally_ different.

#### Read primitive values from objects 

Sometimes, you may receive an object from props:

```JSX
function ChatRoom({ options }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // ✅ All dependencies declared
  // ...
```

The risk here is that the parent component will create the object during rendering:

```JSX
<ChatRoom
  roomId={roomId}
  options={{
    serverUrl: serverUrl,
    roomId: roomId
  }}
/>
```

This would cause your Effect to re-connect every time the parent component re-renders. To fix this, read information from the object _outside_ the Effect, and avoid having object and function dependencies:

```JSX
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

If an object is re-created unintentionally by the parent component, the chat would not re-connect. However, if `options.roomId` or `options.serverUrl` really are different, the chat would re-connect.

## Recap 

- Dependencies should always match the code.
- When you’re not happy with your dependencies, what you need to edit is the code.
- Suppressing the linter leads to very confusing bugs, and you should always avoid it.
- To remove a dependency, you need to “prove” to the linter that it’s not necessary.
- If some code should run in response to a specific interaction, move that code to an event handler.
- If different parts of your Effect should re-run for different reasons, split it into several Effects.
- If you want to update some state based on the previous state, pass an updater function.
- If you want to read the latest value without “reacting” it, extract an Effect Event from your Effect.
- In JavaScript, objects and functions are considered different if they were created at different times.
- Try to avoid object and function dependencies. Move them outside the component or inside the Effect.