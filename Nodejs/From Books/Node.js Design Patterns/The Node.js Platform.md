---
title: The Node.js Platform
created: 2026-11-01
tags:
  - nodejs
---
There are basically two types of I/O. One is blocking and other is non-blocking. In blocking I/O, the function call corresponding to an I/O request will block the execution of the thread until the operation completes. 

In non-blocking I/O, the system call returns immediately without waiting for data to be read or written. Instead we will just use a for loop which will check again and again. If no data is returned we will just continue. If data is present consume it, and if we need to close the connection, close it. This entire process is called **busy-waiting**.

```js
resources = [socketA, socketB, fileA]
while (!resources.isEmpty()) {
for (resource of resources) {
	// try to read
    data = resource.read()
	if (data === NO_DATA_AVAILABLE) {
	// there is no data to read at the moment
	continue
    }
	if (data === RESOURCE_CLOSED) {
		// the resource was closed, remove it from the list
      resources.remove(i)
    } else {
	    // some data was received, process it
	    consumeData(data)
    }
    }
  }
}
```

## Event demultiplexing

There is another strategy of handling concurrent non-blocking resources in an efficient way, **synchronous event demultiplexer** (also known as the **event notification interface**).

Multiplexing(multiple combined into one) and demultiplexing(one broken down into multiple) is the same thing from CN.

The **synchronous event demultiplexer** watches multiple resources and returns a new event when a read or write operation is executed over one of those resources. It is synchronous and therefore blocking until everything from the demultiplexer watch queue is empty.

```js
watchedList.add(socketA, FOR_READ); // (1)
watchedList.add(fileB, FOR_READ);

while ((events = demultiplexer.wait(watchedList))) { // (2)
  // event loop
  for (event of events) { // (3)
    // This read will never block and will always return data
    data = event.resource.read();

    if (data === RESOURCE_CLOSED) {
      // the resource was closed, remove it from the watched list
      demultiplexer.unwatch(event.resource);
    } else {
      // some actual data was received, process it
      consumeData(data);
    }
  }
}
```

In line number (1), resources are added to a data structure, associating each one with a specific operation. In (2), the demultiplexer watches the resources and blocks until any one of them is read for read. When this happens some events are returned which are processed. In (3), we have some event to be processed, the events are ready to be read, and do not block the operation. When all the events are processed, the flow will block again on the event demultiplexer until new events are again available to be processed. This is called the **event loop**.

With this pattern we can handle several I/O operations inside a single thread.

## The reactor pattern 

The main idea behind the reactor pattern is to have a handler associated with each I/O operation. A handler in Node.js is represented by a `callback` (or `cb` for short) function.

The handler will be invoked as soon as an event is produced and processed by the event loop.

![](../../../assets/Pasted%20image%2020261102224202.png)

Fig. The reactor pattern 

This is what happens in an application using the reactor pattern:

1. The application generates a new I/O operation by submitting a request to the **Event Demultiplexer**. The application also specifies a handler, which will be invoked when the operation completes. Submitting a new request to the **Event Demultiplexer** is a non-blocking call and it immediately returns control to the application.
2. When a set of I/O operations completes, the Event Demultiplexer pushes a set of corresponding events into the Event Queue.
3. At this point, the **Event Loop** iterates over the items of the **Event Queue**.
4. For each event, the associated handler is invoked.
5. The handler, which is part of the application code, gives back control to the Event Loop when its execution completes (5a). While the handler executes, it can request new asynchronous operations (5b), causing new items to be added to the Event Demultiplexer (1).
6. When all the items in the Event Queue are processed, the Event Loop blocks again on the Event Demultiplexer, which then triggers another cycle when a new event is available.

A Node.js application will exit when there are no more pending operations in the event demultiplexer, and no more events to be processed inside the event queue.

**The Reactor Pattern**: Handles I/O by blocking until new events are available from a set of observed resources, and then reacts by dispatching each event to an associated handler.

## Libuv, the I/O engine of Node.js

Each OS has its own interface for the event demultiplexer. To create a higher-level abstraction with the objective that Node.js is compatible with all the major operating systems, Libuv was created. 

Other than abstracting the underlying system calls, libuv also implements the reactor pattern, thus providing an API for creating event loops, managing the event queue, running asynchronous I/O operations, and queuing other types of task.

## The recipe for Node.js

The reactor pattern and libuv are the basic building blocks of Node.js, but we need three more components to build the full platform: 
- A set of bindings responsible for wrapping and exposing libuv and other low-level functionalities to JS.
- V8, the JS engine.
- A core JS library that implements the high-level Node.js API. 

# JS in Node.js

JS in browser and server are somewhat different. One difference can be on browser we have things like dom, window which are not present in a server. But server side JS gets access to operating system capabilities.

Node.js can run native code (written in C/C++). Native code is fast and efficient and companies wouldn't have to migrate their legacy codebases. Another benefit is that low level languages can talk to hardware which makes Node.js a popular choice for Internet of things.

Nowadays, most JS virtual machines and also Node.js support WebAssembly (Wasm), a low-level instruction format that allows us to compile languages other than JS (such as C++ or Rust) into a format that is "understandable" by JS VMs. This brings many of the advantage of native code without directly interfering with native code. 