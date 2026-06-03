## What is HTTP?

HTTP (HyperText Transfer Protocol) is the medium through which clients and servers send and receive data.

---

## Core Characteristics

### 1. Stateless

HTTP has no memory of past interactions. Each request carries all the information the server needs to process it. Once the server responds, it forgets the request entirely.

**Benefits of stateless design:**

**Simplicity** — The server doesn't need to store session information, which reduces resource overhead and simplifies architecture.

**Scalability** — Requests can be distributed across multiple servers easily. If one server crashes, client interactions aren't affected because no session state is lost.

Because HTTP is stateless, developers use state management techniques like cookies, sessions, or tokens to maintain continuity across interactions.

### 2. Client-Server Model

```
 ________                     __________
|        |   -- request -->  |          |
| Client |                   |  Server  |
|________|  <-- response --  |__________|
```

**Client** — Typically a browser or application that initiates communication by sending a request. The client is responsible for providing all necessary information such as headers and body.

**Server** — Hosts resources like websites or APIs and waits for incoming requests. When it receives a request, it processes it and sends back an appropriate response.

HTTP states that communication must always be initiated by the client.

---

## HTTP vs HTTPS vs TLS vs SSL

**SSL (Secure Sockets Layer)** was the original protocol for securing communications between a client and server. It encrypts data in transit but has since been deprecated due to known security vulnerabilities.

**TLS (Transport Layer Security)** is the modern, more secure replacement for SSL. It encrypts data in transit, ensuring that anything sent between client and server is protected from interception and tampering. TLS uses certificates to authenticate the server and establish an encrypted connection, preventing eavesdropping and man-in-the-middle attacks.

**HTTPS** is simply HTTP with TLS underneath. The underlying mechanism of HTTPS is TLS, not SSL — though the terms are sometimes used interchangeably.

---

## HTTP Versions

|Version|Key Feature|
|---|---|
|HTTP/1.0|Each request opened a new TCP connection — inefficient|
|HTTP/1.1|Persistent connections, chunked transfer encoding, better caching|
|HTTP/2.0|Multiplexing, binary framing, header compression, server push|
|HTTP/3.0|Built on UDP (via QUIC), faster connection setup, no head-of-line blocking|

**HTTP/1.0** — Every request opened and closed a new TCP connection, creating significant overhead.

**HTTP/1.1** — Introduced persistent connections, allowing multiple requests and responses over a single TCP connection. Also added chunked transfer encoding and improved caching mechanisms.

**HTTP/2.0** — Added multiplexing, allowing multiple requests and responses to travel over a single connection simultaneously. Uses binary framing instead of plain text and supports header compression (HPACK). Server push lets the server send resources before the client asks for them. However, head-of-line blocking is still an issue at the TCP level.

**HTTP/3.0** — Built on top of UDP via a protocol called QUIC rather than TCP. This improves performance with faster connection establishment, reduced latency, and better handling of packet loss. Fully solves head-of-line blocking since QUIC handles streams independently.

---

## HTTP Messages

### Request Message

```
PUT /api/user/12345 HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Authorization: Bearer <token>
Content-Type: application/json
Cookie: session_id=abc123

{ "name": "John" }
```

- `PUT` — HTTP method
- `/api/user/12345` — Resource URL being requested
- `HTTP/1.1` — HTTP version
- `Host: example.com` — The backend domain being targeted
- Lines after the method line until the blank line — Request headers
- After the blank line — Request body

### Response Message

```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 256
Cache-Control: max-age=3600

{ "id": 12345, "name": "John" }
```

- `HTTP/1.1` — HTTP version
- `200 OK` — Status code
- Lines after the status line until the blank line — Response headers
- After the blank line — Response body

---

## HTTP Headers

Headers are key-value pairs that carry metadata about a request or response.

**Why headers and not just the URL or body?** The URL is meant to identify a resource, and the body carries the data payload. Headers serve a distinct purpose: they carry metadata and control instructions — things like what format is acceptable, who the client is, how to cache the response, and security policies. Mixing this metadata into the URL would make it unwieldy and expose sensitive information. Putting it in the body would conflate transport concerns with data concerns. Headers keep these concerns cleanly separated.

### Request Headers

|Header|Purpose|
|---|---|
|`User-Agent`|Identifies the client (browser, Postman, etc.)|
|`Authorization`|Carries credentials like tokens for server-side identity verification|
|`Accept`|Tells the server what content types the client can handle|
|`Accept-Language`|Preferred language for the response|
|`Accept-Encoding`|Compression formats the client supports (gzip, br, etc.)|

### General Headers (used in both requests and responses)

|Header|Purpose|
|---|---|
|`Date`|Timestamp of the message|
|`Cache-Control`|Directives for caching behavior|
|`Connection`|Controls whether the connection stays open|

### Representation Headers

These describe the format or encoding of the resource being transferred.

|Header|Purpose|
|---|---|
|`Content-Type`|Media type of the body (e.g. `application/json`)|
|`Content-Length`|Size of the body in bytes|
|`Content-Encoding`|Encoding applied to the body (e.g. gzip)|
|`ETag`|Unique identifier for a resource version, primarily used for caching|
|`Last-Modified`|Timestamp of when the resource was last changed|

### Security Headers

|Header|Purpose|
|---|---|
|`Strict-Transport-Security`|Forces the client to communicate over HTTPS only, preventing protocol downgrade attacks|
|`Content-Security-Policy`|Restricts sources from which JS, CSS, etc. can be loaded — prevents XSS attacks|
|`X-Frame-Options`|Prevents the page from being embedded in an iframe, mitigating clickjacking|
|`X-Content-Type-Options`|Prevents the browser from guessing MIME types, stopping MIME sniffing attacks|
|`Set-Cookie`|Sets cookies with `HttpOnly` or `Secure` flags to ensure they travel only over HTTPS|

### Why Headers Matter

**Extensibility** — HTTP is highly extensible because headers can be added or customized without changing the underlying protocol. Developers can even define custom headers to alter behavior without touching the core spec.

**Content negotiation** — Headers like `Accept`, `Accept-Language`, and `Accept-Encoding` let the server serve different versions of content based on what the client can handle.

**Remote control** — Headers act like a remote control on the server. Clients can specify preferred formats, control caching, pass credentials for access control, and more — all without changing the resource URL.

---

## HTTP Methods

Methods exist to represent the different types of actions a client can request on a server. Instead of every request doing the same thing, the method declares the intent.

|Method|Purpose|Has Body|
|---|---|---|
|`GET`|Retrieve a resource. Must not modify anything on the server|No|
|`POST`|Create a new resource on the server|Yes|
|`PUT`|Replace an existing resource entirely with the request body|Yes|
|`PATCH`|Partially update a resource — prefer this over PUT for partial updates|Yes|
|`DELETE`|Remove a resource from the server|No|
|`OPTIONS`|Inquire about what methods/headers the server supports for a route|No|

### Idempotent vs Non-Idempotent

An **idempotent** method means calling it multiple times with the same input produces the same result.

**Idempotent:** `GET`, `PUT`, `DELETE`

- `GET` — Reads a resource without modifying it. Same result every time.
- `PUT` — Completely replaces a resource. No matter how many times you replace old data with the same new data, the result is always that new data.
- `DELETE` — After the first call, the resource is gone. Subsequent calls either get 404 or no-op — the server state doesn't keep changing.

**Non-Idempotent:** `POST`

- `POST` — Creates a new resource on each call. Sending the same body twice results in two separate resources being created.

`PATCH` is technically non-idempotent in the general case (e.g. "increment counter by 1"), though in practice most PATCH implementations are written to be idempotent.

---

## CORS and the OPTIONS Method

### Same-Origin Policy

Browsers enforce a same-origin policy: a page on `example.com` cannot make requests to `api.anotherdomain.com` unless that server explicitly allows it. This is where CORS (Cross-Origin Resource Sharing) comes in.

### Simple Requests

A simple request is sent directly to the server without a preflight. The browser attaches an `Origin` header and the server responds with `Access-Control-Allow-Origin`. The browser checks that header — if it's `*` or matches the frontend origin, the response is passed through. Otherwise it's blocked.

```
GET /api/products/123 HTTP/1.1
Host: api.anotherdomain.com
Origin: https://example.com
Accept: application/json
```

### Preflight Requests

For requests that might have side effects, the browser first sends an OPTIONS request to ask the server whether the real request is allowed.

**A request triggers a preflight if any of the following are true (and it's cross-origin):**

1. The method is not `GET`, `POST`, or `HEAD` (e.g. `PUT`, `DELETE`)
2. It includes non-simple headers like `Authorization` or custom headers
3. The `Content-Type` is not `application/x-www-form-urlencoded`, `multipart/form-data`, or `text/plain`

**Preflight request:**

```
OPTIONS /api/resource HTTP/1.1
Host: api.example.com
Origin: http://example.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: Authorization
```

**Preflight response (if server is configured correctly):**

```
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://example.com
Access-Control-Allow-Methods: PUT, DELETE
Access-Control-Allow-Headers: Authorization
Access-Control-Max-Age: 86400
```

- `Access-Control-Allow-Origin` — Which origins are permitted
- `Access-Control-Allow-Methods` — Which methods are supported on this route
- `Access-Control-Allow-Headers` — Which headers are permitted
- `Access-Control-Max-Age` — How long (in seconds) this preflight result can be cached — `86400` means 24 hours, so the browser won't repeat the preflight for a day

---

## HTTP Status Codes

Status codes exist to communicate the result of a request in a standardized way. Without them, clients would have to guess outcomes from the response body, leading to inconsistencies. Status codes provide a universal language all clients and servers understand.

### 1xx — Informational

Used to communicate interim responses before the final response.

- `100 Continue` — Server received the headers and the client should proceed to send the request body. Useful for large uploads: the client sends headers first; if the server is okay with them, it sends 100 before the client sends the potentially large body.
- `101 Switching Protocols` — Server agrees to switch protocols as requested (e.g. upgrading from HTTP to WebSockets).

### 2xx — Success

- `200 OK` — Request was successful. Server is returning the requested resource or has performed the action.
- `201 Created` — Request was fulfilled and a new resource was created.
- `204 No Content` — Request succeeded but there is no body to return. Common in preflight responses and DELETE operations.

### 3xx — Redirection

- `301 Moved Permanently` — Resource has been permanently moved to a new URL. Future requests should use the new URL.
- `302 Found (Temporary Redirect)` — Resource is temporarily at a different URL but the client should continue using the original URL in future.
- `304 Not Modified` — Used with conditional requests and caching. The resource hasn't changed since the client last fetched it, so the client should use its cached copy.

### 4xx — Client Errors

- `400 Bad Request` — The client sent invalid or illogical data.
- `401 Unauthorized` — The client is not authenticated. Credentials are missing or invalid.
- `403 Forbidden` — The server understood the request but refuses to fulfill it. This can happen to authenticated users who lack permission to access a resource.
- `404 Not Found` — The requested resource doesn't exist.
- `405 Method Not Allowed` — The HTTP method used is not supported for this route.
- `409 Conflict` — Request conflicts with the current state of the server (e.g. trying to create a resource with a name that must be unique).
- `429 Too Many Requests` — The client has sent too many requests in a given time (rate limiting).

### 5xx — Server Errors

- `500 Internal Server Error` — An unexpected condition was encountered on the server. Often returned as a generic error to avoid leaking server internals to the client.
- `501 Not Implemented` — The server doesn't support the requested HTTP method or functionality but may in the future.
- `502 Bad Gateway` — The server, acting as a gateway or proxy, received an invalid response from an upstream server.
- `503 Service Unavailable` — The server is temporarily unable to handle requests (overloaded or down for maintenance).
- `504 Gateway Timeout` — The server, acting as a gateway, didn't receive a timely response from an upstream server.

---

## HTTP Caching

Caching stores copies of responses for reuse, reducing repeated requests to the server. This reduces load times, bandwidth usage, and server load.

**How it works:**

When the server responds, it includes caching instructions:

```
Cache-Control: max-age=3600, public
ETag: "abc123"
Last-Modified: Tue, 03 Jun 2025 10:00:00 GMT
```

On subsequent requests for the same resource, the client sends:

```
If-None-Match: "abc123"
If-Modified-Since: Tue, 03 Jun 2025 10:00:00 GMT
```

The server checks: if the ETag still matches and the resource hasn't been modified since that timestamp, it returns `304 Not Modified` with no body — the client uses its cached copy. Otherwise it sends the full updated resource with a new ETag.

**Cache-Control directives:**

- `max-age=N` — Cache the response for N seconds
- `public` — Response can be cached by any cache (browsers, CDNs)
- `private` — Response can only be cached by the individual user's browser
- `no-cache` — Must revalidate with the server before using cached copy
- `no-store` — Do not cache at all

---

## Content Negotiation

A mechanism by which client and server agree on the best format to exchange data. The client indicates its preferences and the server responds in a compatible format, or falls back to a default.

**Types:**

1. **Media type negotiation** — Client specifies desired format via the `Accept` header. Example: `Accept: application/json, text/html`
2. **Language negotiation** — Client requests a specific language via `Accept-Language`. Example: `Accept-Language: en-US, hi`
3. **Encoding negotiation** — Client specifies supported compression formats via `Accept-Encoding`. Example: `Accept-Encoding: gzip, br, deflate`

---

## HTTP Compression

Compression reduces the size of response bodies before they are sent over the network, improving transfer speed and reducing bandwidth.

**How it works:** The client advertises supported compression algorithms via the `Accept-Encoding` header. The server compresses the response body using one of those algorithms and declares which one it used in the `Content-Encoding` response header.

**Common compression formats:**

- **gzip** — The most widely supported format. Uses the DEFLATE algorithm. Good compression ratio with fast decompression. Supported by virtually all browsers and servers.
- **br (Brotli)** — Developed by Google. Achieves better compression than gzip (typically 15–25% smaller) at comparable speeds. Ideal for text-based content like HTML, CSS, and JS. Supported by all modern browsers. Only works over HTTPS.
- **deflate** — The raw DEFLATE algorithm. Less commonly used than gzip because of inconsistent implementation across servers and clients.
- **zstd (Zstandard)** — Newer format from Facebook. Excellent compression ratio with very fast decompression. Gaining support in modern browsers and servers.

**What gets compressed:** Text-based content — HTML, CSS, JavaScript, JSON, XML — benefits the most. Binary formats like images, videos, and already-compressed files (ZIP, PDF) generally should not be compressed as they're already compressed and the overhead isn't worth it.

---

## Persistent Connections and Keep-Alive

In HTTP/1.0, each request-response cycle required opening and closing a new TCP connection — expensive and slow.

HTTP/1.1 introduced **persistent connections**: a single TCP connection is reused for multiple request-response cycles, eliminating the overhead of repeatedly establishing connections.

**Keep-Alive header:**

```
Connection: keep-alive
Keep-Alive: timeout=5, max=100
```

- `timeout=5` — Keep the connection open for up to 5 seconds of inactivity
- `max=100` — Allow up to 100 requests over this connection before closing it

In HTTP/1.1, connections are persistent by default. The `keep-alive` header can still be used to negotiate specific timeout and max-request parameters. The connection closes when either the server or client sends `Connection: close`, or when the timeout or max limit is reached.

HTTP/2 and HTTP/3 make this largely irrelevant since they handle multiplexing at the protocol level — multiple requests fly over a single connection simultaneously without needing any keep-alive negotiation.

---

## Handling Large Requests and Responses

### Multipart Requests (Client → Server)

Used when a client needs to send large files or multiple pieces of data in one request. The file's binary data is split into parts, each separated by a **boundary** delimiter specified in the `Content-Type` header.

```
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW

------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="file"; filename="photo.jpg"
Content-Type: image/jpeg

<binary data here>
------WebKitFormBoundary7MA4YWxkTrZu0gW--
```

### Chunked Transfer / Streaming (Server → Client)

Used when the server sends a large response incrementally rather than all at once. The server sends multiple chunks, each containing a portion of the data.

```
Content-Type: text/event-stream
Transfer-Encoding: chunked
Connection: keep-alive
```

This is commonly used in:

- Server-Sent Events (SSE) for real-time data
- AI/LLM streaming responses
- Large file downloads where the total size isn't known upfront