---
title: HTTP handling in nodejs
source: https://nodejs.org/learn/http/anatomy-of-an-http-transaction
created: 2026-07-11
tags:
  - javascript
  - nodejs
---
## Create the server

You can create a web server object using `createServer`:

```js
import http from 'node:http';
const server = http.createServer((request, response) => {
  // magic happens here!
});
```

The function that's passed in to `createServer` is called once for every HTTP request that's made against that server, so it's called the request handler. The server object returned by `createServer` is an `EventEmitter`.

Node.js core API is built around an idiomatic asynchronous event-driven architecture in which certain kinds of objects (called 'emitters') emit named events that cause `Function`  objects ('listeners') to be called. All objects that emit events are instances of the `EventEmitter` class.

We could have also wrote the above thing as:

```js
const server = http.createServer();
server.on('request', (request, response) => {
  // the same kind of magic happens here!
});
```

When an HTTP request hits the server, Node calls the request handler function with a few handy objects for dealing with the transaction, `request` and `response`.

In most cases, we just need to `listen` on the `server` and mention a port number.

## Method, URL and Headers

We can access `method` and `url` by destructuring the `request` object. 

```js
const { method, url } = request;
```

_Note: The `request` object is an instance of `IncomingMessage`._

Headers are in their own object on `request` called `headers`.

```js
const { headers } = request;
const userAgent = headers['user-agent'];
```

Headers are represented in lower-case only, no matter how client sends them.

If some headers are repeated, then their values are overwritten or joined together as comma-separated strings.

_Note: If we need headers as they were passed by the client, we can access them using `request.rawHeaders`._

## Request Body

The `request` object that's passed in to handler implements the `ReadableStream` interface. This stream can be listened to or piped elsewhere just like any other stream. We can grab the data right out of the stream by listening to the stream's `'data'` and `'end'` events. 

The chunk emitted in each `'data'` event is a `Buffer`. If we know it's going to be string data, the best thing to do is collect the data in an array, then at the `'end'`, concatenate and stringify it. 

```js
let body = [];
request
  .on('data', chunk => {
    body.push(chunk);
  })
  .on('end', () => {
    body = Buffer.concat(body).toString();
    // at this point, `body` has the entire request body stored in it as a string
  });
```

## Errors

Since the `request` object is a `ReadableStream`, it's also an `EventEmitter` and behaves like one when an error happens.

An error in the `request` stream presents itself by emitting an `'error'` event on the stream. **If you don't have a listener for that event, the error will be _thrown_, which could crash your Node.js program.** You should therefore add an `'error'` listener on your request streams.

```js
request.on('error', err => {
  // This prints the error message and stack trace to `stderr`.
  console.error(err.stack);
});
```

So far we can read the data presented to us, sense errors and read url and methods along with headers.

```js
import http from 'node:http';
http
  .createServer((request, response) => {
    const { headers, method, url } = request;
    let body = [];
    request
      .on('error', err => {
        console.error(err);
      })
      .on('data', chunk => {
        body.push(chunk);
      })
      .on('end', () => {
        body = Buffer.concat(body).toString();
        // At this point, we have the headers, method, url and body, and can now
        // do whatever we need to in order to respond to this request.
      });
  })
  .listen(8080); // Activates this server, listening on port 8080.
```

## HTTP Status Code

_Note: `response` object is an instance of `ServerResponse`, which is a `WritableStream`._

If we don't set it, the HTTP status code on a response will always be 200. To set it manually, we can use the `statusCode` property.

```js
response.statusCode = 404; // Tell the client that the resource wasn't found.
```

## Setting Response Headers

Headers are set through a convenient method called `setHeader`.

```js
response.setHeader('Content-Type', 'application/json');
response.setHeader('X-Powered-By', 'bacon');
```

When setting the headers on a response, the case is insensitive on their names. If we set a header repeatedly, the last value you set is the value that gets sent. 

## Explicitly Sending Header Data

The methods of setting the headers and status code that we've already discussed assume that we're using "implicit headers". This means we're counting on node to send that headers for us at the correct time before sending the body data. 

If we want, we can _explicitly_ write the headers to the response stream. 

```js
response.writeHead(200, {
  'Content-Type': 'application/json',
  'X-Powered-By': 'bacon',
});
```

`writeHead` write both status code and the headers to the stream. 

Once we've set the headers (either _implicitly_ or _explicitly_), we're ready to start sending response data.

## Sending Response Body

Since the `response` object is a `WritableStream`, writing a response body out to the client is just a matter of using the usual stream methods.

```js
response.write('<html>');
response.write('<body>');
response.write('<h1>Hello, World!</h1>');
response.write('</body>');
response.write('</html>');
response.end();
```

The `end` function on streams can also take in some optional data to send as the last bit of data on the stream, so we can simplify the example above as follows.

```js
response.end('<html><body><h1>Hello, World!</h1></body></html>');
```

_Note: It's important to set the status and headers before we start writing chunks of data to the body. This makes sense, since headers come before the body in HTTP responses._

## More about Errors

The `response` stream can also emit `'error'` events. and at some point we're going to have to deal with them as well. 

## Putting it all together

```js
const http = require('node:http');
http
  .createServer((request, response) => {
    const { headers, method, url } = request;
    let body = [];
    request
      .on('error', err => {
        console.error(err);
      })
      .on('data', chunk => {
        body.push(chunk);
      })
      .on('end', () => {
        body = Buffer.concat(body).toString();
        // BEGINNING OF NEW STUFF
        response.on('error', err => {
          console.error(err);
        });
        response.statusCode = 200;
        response.setHeader('Content-Type', 'application/json');
        // Note: the 2 lines above could be replaced with this next one:
        // response.writeHead(200, {'Content-Type': 'application/json'})
        const responseBody = { headers, method, url, body };
        response.write(JSON.stringify(responseBody));
        response.end();
        // Note: the 2 lines above could be replaced with this next one:
        // response.end(JSON.stringify(responseBody))
        // END OF NEW STUFF
      });
  })
  .listen(8080);
```

We can even customize above code so that we can send different response according to different method (`request.method`) or url (`request.url`).

We have learned that the `request` object is a `ReadableStream` and the `response` object is a `WrittableStream`. That means we can use `pipe` to direct data from one to the other. 

Here's an example with `pipe` and error handling.

```js
import http from 'node:http';
http
  .createServer((request, response) => {
    request.on('error', err => {
      console.error(err);
      response.statusCode = 400;
      response.end();
    });
    response.on('error', err => {
      console.error(err);
    });
    if (request.method === 'POST' && request.url === '/echo') {
      request.pipe(response);
    } else {
      response.statusCode = 404;
      response.end();
    }
  })
  .listen(8080);
```

## Further reading

- [Events](https://nodejs.org/api/events.html)
- [Stream](https://nodejs.org/api/stream.html)
- [HTTP](https://nodejs.org/api/http.html)