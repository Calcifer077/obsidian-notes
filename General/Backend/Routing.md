---
title: Routing
source: https://youtu.be/SubuU1iOC2s?si=zj-PEoxxMTCxrGw6
created: 2026-06-09
---
Routing is about telling the server **where you want to go** — which resource you want to access and what action you want to perform on it.

```
GET /users → [{ }, { }, { }]
```

`GET` is the action (fetch data), `/users` is the route (where that data lives). The server takes the combination of action + route and maps it to a specific handler, which runs the business logic, performs database operations, and returns a response.

A route can accept multiple methods and produce entirely different results for each, because each method maps to a different handler.

```
GET    /users   → returns list of users
POST   /users   → creates a new user
```

---

### Route Parameters (Path Parameters)

**Static routes** have no variable parts — `/api/books` is always `/api/books`.

**Dynamic routes** contain variable segments. `/api/books/:id` means the server matches `/api/books/` followed by any value. That value (`:id`) is called a **route parameter** or **path parameter**.

```
GET /api/books/123   → :id = 123
GET /api/books/456   → :id = 456
```

---

### Query Parameters

Query parameters are key-value pairs appended to the URL after a `?`. They're used to send metadata or filtering instructions to the server rather than identifying a specific resource.

```
GET /api/search?query=javascript&page=2&limit=10
```

Common uses: search queries, pagination, filtering, sorting.

---

### Nested Routes

Not a distinct routing type — more of a design pattern for expressing relationships between resources.

```
GET /api/users              → all users
GET /api/users/123          → user with id 123
GET /api/users/123/posts    → all posts by user 123
GET /api/users/123/posts/143 → post 143 by user 123
```

---

### API Versioning

When new requirements force breaking changes to your API, you create a new version rather than modifying the existing one — so existing clients aren't broken.

```
GET /api/v1/users   → old format (still works for old clients)
GET /api/v2/users   → new format
```

You can deprecate older versions gradually, giving clients time to migrate.

---

### Catch-All Routes

A wildcard route (`*`) matches any path that hasn't been explicitly defined. Instead of returning an error for unimplemented routes, the server can respond with a clean 404 page or a helpful message.

```
*  → "Route not found"
```