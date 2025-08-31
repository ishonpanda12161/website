# Routing Specification

Hono provides a flexible and powerful routing system that supports various path patterns, HTTP methods, and advanced routing techniques.

## Route Registration

### HTTP Method Routes

Register handlers for specific HTTP methods.

#### `get(path, ...handlers)`

#### `post(path, ...handlers)`

#### `put(path, ...handlers)`

#### `delete(path, ...handlers)`

#### `head(path, ...handlers)`

#### `options(path, ...handlers)`

#### `patch(path, ...handlers)`

**Parameters:**

- `path`: `string` - Route path pattern
- `...handlers`: `Handler[]` - Request handlers and middleware

**Returns:** `Hono` - The Hono instance for method chaining

#### `all(path, ...handlers)`

Register handlers for all HTTP methods.

**Parameters:**

- `path`: `string` - Route path pattern
- `...handlers`: `Handler[]` - Request handlers and middleware

**Returns:** `Hono`

#### `on(method, path, ...handlers)`

Register handlers for custom or multiple HTTP methods.

**Parameters:**

- `method`: `string | string[]` - HTTP method(s) to handle
- `path`: `string | string[]` - Route path pattern(s)
- `...handlers`: `Handler[]` - Request handlers and middleware

**Returns:** `Hono`

**Notes:**

- Supports custom HTTP methods like `PURGE`, `PATCH`
- Can handle multiple methods with a single registration
- Can handle multiple paths with a single registration

## Path Patterns

### Static Paths

Exact path matching for static routes.

**Format:** `/exact/path`

**Examples:**

- `/users` - Matches only `/users`
- `/api/v1/status` - Matches only `/api/v1/status`

### Path Parameters

Dynamic path segments that capture values.

**Format:** `/path/:parameter`

**Parameter Access:** `c.req.param('parameter')`

**Notes:**

- Parameters are available in handlers via `c.req.param()`
- Parameter names must be valid JavaScript identifiers
- Parameters are always strings

### Optional Parameters

Path segments that may or may not be present.

**Format:** `/path/:parameter?`

**Notes:**

- Routes match both `/path` and `/path/value`
- Optional parameters return `undefined` when not present

### Wildcards

Match any path segment or multiple segments.

**Single Segment:** `/path/*`  
**Multiple Segments:** `/path/*` (matches `/path/a/b/c`)

**Notes:**

- `*` matches one or more path segments
- Greedy matching - captures everything remaining in path

### Regular Expression Patterns

Constrain parameter values using regular expressions.

**Format:** `/path/:param{regex}`

**Examples:**

- `/posts/:id{[0-9]+}` - Numeric IDs only
- `/posts/:slug{[a-z-]+}` - Lowercase letters and hyphens
- `/files/:filename{.+\\.pdf}` - Files ending with .pdf

**Notes:**

- Regex patterns are enclosed in curly braces
- Invalid patterns will not match the route
- Use double backslashes for literal backslashes

## Route Organization

### Method Chaining

Chain multiple route registrations on the same path.

**Returns:** `Hono` instance for continued chaining

**Benefits:**

- Cleaner code organization
- Reduced repetition
- Better readability for related routes

### Route Grouping

Organize routes into logical groups using sub-applications.

#### `route(path, subApp)`

Mount a sub-application at the specified path.

**Parameters:**

- `path`: `string` - Mount point for the sub-application
- `subApp`: `Hono` - Sub-application instance

**Returns:** `Hono`

**Notes:**

- All routes in `subApp` are prefixed with `path`
- Middleware in `subApp` is preserved
- Sub-applications can be nested

#### `basePath(path)`

Set a base path for all subsequent routes.

**Parameters:**

- `path`: `string` - Base path to prepend

**Returns:** `Hono<E, S, BasePath>` - New typed instance

**Notes:**

- Returns a new Hono instance with updated type
- All routes registered after this call are prefixed
- Does not affect previously registered routes

## Advanced Routing

### Hostname-Based Routing

Route based on the request hostname.

**Configuration:**

```ts
const app = new Hono({
  getPath: (req) => req.url.replace(/^https?:\/\/([^?]+).*$/, '$1'),
})
```

**Route Registration:**

- Routes include the hostname: `app.get('/api.example.com/data', handler)`

### Host Header Routing

Route based on the `Host` header value.

**Configuration:**

```ts
const app = new Hono({
  getPath: (req) =>
    '/' +
    req.headers.get('host') +
    req.url.replace(/^https?:\/\/[^/]+(\/[^?]*).*/, '$1'),
})
```

### Custom Path Extraction

Override path extraction for custom routing logic.

**Type:** `(req: Request) => string`

**Use Cases:**

- Hostname-based routing
- Header-based routing
- Custom URL schemes
- Multi-tenant applications

## Route Matching

### Priority Rules

Routes are matched in registration order with specific precedence:

1. **Exact static paths** - Highest priority
2. **Static paths with parameters** - By registration order
3. **Wildcard paths** - Lowest priority

**Best Practice:** Register more specific routes before general ones

### Execution Behavior

- **First match wins** - Processing stops at first matching handler
- **Middleware continues** - Middleware calls `next()` to continue
- **Handler stops** - Regular handlers stop the matching process

### Route Conflicts

When multiple routes could match the same request:

- Routes are tested in registration order
- First matching route handles the request
- Later routes are never evaluated

## Error Handling

### Route Not Found

When no routes match a request:

- Default: Returns 404 status
- Customizable via `app.notFound(handler)`
- Handler receives the context object

### Invalid Patterns

- Invalid regex patterns in path parameters will not match
- Malformed route patterns may cause runtime errors
- Always test route patterns with expected input

## Type Safety

### Path Parameter Types

Path parameters are typed based on the route pattern:

```ts
app.get('/users/:id', (c) => {
  const id = c.req.param('id') // string | undefined
})
```

### Route Schema Integration

When using schema validation:

```ts
type Schema = {
  '/users/:id': {
    param: { id: string }
  }
}

const app = new Hono<{}, Schema>()
```

## Performance Considerations

### Router Selection

Different routers optimize for different scenarios:

- **SmartRouter** (default) - Best overall performance
- **RegExpRouter** - Good for complex patterns
- **TrieRouter** - Memory efficient for many routes
- **LinearRouter** - Simple, good for few routes

### Route Organization

- **Group related routes** - Better router optimization
- **Static routes first** - Faster matching
- **Avoid complex regex** - Can impact performance

## See Also

- [Hono Specification](/docs/api/hono) - Application instance methods
- [Context Specification](/docs/api/context) - Request handling context
- [Routing Examples](/docs/api/routing-examples) - Practical routing patterns
- [Presets Specification](/docs/api/presets) - Router configuration options
