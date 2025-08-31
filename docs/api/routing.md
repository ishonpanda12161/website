# Routing Specification

Hono's routing system matches HTTP requests to registered handlers based on URL path patterns and HTTP methods. It supports path parameters, wildcards, regular expressions, and route composition.

## Route Definition

### Basic Route Registration

Routes are registered using HTTP method functions on a Hono app instance:

```ts
app.get(path, ...handlers)
app.post(path, ...handlers)
app.put(path, ...handlers)
app.delete(path, ...handlers)
app.patch(path, ...handlers)
app.options(path, ...handlers)
app.head(path, ...handlers)
```

Parameters:
- path: `string | string[]` — URL path pattern(s) to match
- handlers: `...MiddlewareHandler` — Handler functions for the route

### Generic Route Registration

```ts
app.on(method, path, ...handlers)
```

Parameters:
- method: `string | string[]` — HTTP method(s) (e.g., 'PURGE', ['PUT', 'DELETE'])
- path: `string | string[]` — Path pattern(s)
- handlers: `...MiddlewareHandler` — Handler functions

```ts
app.all(path, ...handlers)
```
- Registers handlers for all HTTP methods for the given path(s)

## Path Patterns

### Static Paths

Exact string matching:
- `/hello` — Matches only `/hello`
- `/api/users` — Matches only `/api/users`

### Path Parameters

Named parameters captured as variables:

```ts
app.get('/user/:name', (c) => {
  const name = c.req.param('name') // Extract parameter value
})
```

- `:name` — Required parameter
- `:name?` — Optional parameter (matches both `/api/animal` and `/api/animal/cat`)

### Wildcards

- `*` — Matches any path segment: `/wild/*/card` matches `/wild/anything/card`
- `**` — Matches multiple path segments: `/files/**` matches `/files/a/b/c`

### Regular Expression Constraints

Parameters can include regex patterns:

```ts
app.get('/post/:date{[0-9]+}/:title{[a-z]+}', handler)
```

- `{[0-9]+}` — Digits only
- `{[a-z]+}` — Lowercase letters only  
- `{.+\.png}` — Includes slashes: matches paths like `image/file.png`

## Route Composition

### Route Grouping

Combine multiple routes under a common prefix:

```ts
const api = new Hono()
api.get('/users', handler)
api.post('/users', handler)

app.route('/api', api) // Mounts at /api/users, /api/users (POST)
```

### Base Path

Set a base path for all routes in a Hono instance:

```ts
const api = new Hono().basePath('/api')
api.get('/users', handler) // Accessible at /api/users
```

### Method Chaining

Chain multiple methods for the same path:

```ts
app
  .get('/endpoint', getHandler)
  .post(postHandler)     // Same path as above
  .delete(deleteHandler) // Same path as above
```

## Registration Order and Priority

Routes are matched in **registration order**:

1. **First match wins**: Once a route matches and executes, processing stops
2. **More specific routes first**: Register specific routes before generic ones
3. **Middleware before handlers**: Use `app.use()` before route handlers
4. **Fallback routes last**: Place wildcard routes after specific routes

```ts
// Correct order
app.get('/api/users/admin', specificHandler)  // Most specific first
app.get('/api/users/:id', paramHandler)      // Parameter routes next  
app.get('/api/*', fallbackHandler)           // Wildcard routes last
```

## Advanced Routing

### Hostname Routing

Include hostname in path matching:

```ts
const app = new Hono({
  getPath: (req) => req.url.replace(/^https?:\/([^?]+).*$/, '$1')
})

app.get('/www1.example.com/hello', handler)
```

### Header-based Routing

Route based on header values:

```ts
const app = new Hono({
  getPath: (req) => '/' + req.headers.get('host') + req.url.replace(/^https?:\/\/[^/]+(\/[^?]*).*/, '$1')
})
```

## Caveats and Edge Cases

### Strict Mode
- Default: `/hello` and `/hello/` are different routes
- With `strict: false`: both paths are treated the same

### Route Composition Order
Routes must be composed in correct order:

```ts
// ✅ Correct order
subRouter.get('/endpoint', handler)
app.route('/api', subRouter)

// ❌ Wrong order - results in 404
app.route('/api', subRouter) // subRouter has no routes yet
subRouter.get('/endpoint', handler) // Added after mounting
```

### Multiple Path/Method Arrays
Both paths and methods can be arrays:

```ts
app.on(['GET', 'POST'], ['/hello', '/hi'], handler)
// Handles: GET /hello, POST /hello, GET /hi, POST /hi
```

## See Also

- [Hono Object](/docs/api/hono) — App instance and route registration methods
- [Context](/docs/api/context) — Request context and parameter extraction  
- [Request](/docs/api/request) — Path parameter and query string access