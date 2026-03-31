`route.js` is a special file used in the **App Router** to create **Route Handlers**, allowing us to define custom backend API endpoints (e.g., `/api/user`) that act on HTTP methods like GET, POST, PUT, and DELETE. Unlike `page.js` which returns UI, `route.js` handled server-side logic using standard Web Request/Response APIs.

You can't have both `page.js` and `route.js` on the same route.

##### HTTP methods
A **route** file allows us to create custom request handlers for a given route. The following [HTTP methods](https://developer.mozilla.org/docs/Web/HTTP/Methods) are supported: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD`, and `OPTIONS`.
Usage:
```js
export async function GET(request) {}
 
export async function HEAD(request) {}
 
export async function POST(request) {}
 
export async function PUT(request) {}
 
export async function DELETE(request) {}
 
export async function PATCH(request) {}
 
export async function OPTIONS(request) {}
```

##### Parameters
**`request` (optional)**
The `request` object is a [NextRequest](https://nextjs.org/docs/app/api-reference/functions/next-request) object, which is an extension of the Web [Request](https://developer.mozilla.org/docs/Web/API/Request) API. `NextRequest` gives you further control over the incoming request, including easily accessing `cookies` and an extended, parsed, URL object `nextUrl`.

Example:
```js
export async function GET(request) {
  const url = request.nextUrl
}
```

**`context` (optional)**
- **`params`**: a promise that resolves to an object containing the [dynamic route parameters](https://nextjs.org/docs/app/api-reference/file-conventions/dynamic-routes) for the current route.

Example:
```js
export async function GET(request, { params }) {
  const { team } = await params
}
```

You can send response using `Response`.
Example:
```js
return new Response('Hello, Next.js!', { status: 200, headers: { 'Set-Cookie': `token=${token.value}` }, })
```

Read more:
[Next.js route](https://nextjs.org/docs/app/api-reference/file-conventions/route)
