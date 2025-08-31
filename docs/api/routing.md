# Routing System Specification

The Hono routing system provides flexible and intuitive URL pattern matching with support for parameters, wildcards, regex constraints, and route organization.

## Route Registration Methods

All routing methods return the Hono instance for method chaining.

### HTTP Method Routes

#### `.get(path, ...handlers)`
#### `.post(path, ...handlers)`
#### `.put(path, ...handlers)`
#### `.delete(path, ...handlers)`
#### `.patch(path, ...handlers)`
#### `.head(path, ...handlers)`
#### `.options(path, ...handlers)`

Register routes for specific HTTP methods.

**Parameters:**
- `path`: `string` - URL pattern to match
- `...handlers`: `Handler[]` - Route handlers and middleware

**Returns:** `Hono` - The application instance (for chaining)

### Multi-Method Routes

#### `.all(path, ...handlers)`

Register route for all HTTP methods.

**Parameters:**
- `path`: `string` - URL pattern to match  
- `...handlers`: `Handler[]` - Route handlers and middleware

**Returns:** `Hono` - The application instance

#### `.on(methods, paths, ...handlers)`

Register route for specific methods and/or paths.

**Parameters:**
- `methods`: `HTTPMethod | HTTPMethod[]` - HTTP method(s) to match
- `paths`: `string | string[]` - URL pattern(s) to match
- `...handlers`: `Handler[]` - Route handlers and middleware

**Returns:** `Hono` - The application instance

**HTTPMethod Types:**
- Standard: `'GET'`, `'POST'`, `'PUT'`, `'DELETE'`, `'PATCH'`, `'HEAD'`, `'OPTIONS'`
- Custom: Any string (e.g., `'PURGE'`, `'CONNECT'`)

## Path Pattern Syntax

### Static Paths

Exact string matching for URL segments.

**Pattern:** `/users/profile`
**Matches:** `/users/profile`
**Does not match:** `/users/profile/`, `/users/profiles`

### Path Parameters

#### Named Parameters

**Pattern:** `/:name`
**Description:** Captures a single URL segment as a named parameter
**Matches:** Any non-empty segment except `/`

**Examples:**
- `/user/:id` matches `/user/123`, `/user/john`
- `/posts/:slug/comments` matches `/posts/hello-world/comments`

#### Optional Parameters

**Pattern:** `/:name?`
**Description:** Parameter that may or may not be present

**Examples:**
- `/api/animal/:type?` matches `/api/animal` and `/api/animal/cat`

#### Regex Constrained Parameters

**Pattern:** `/:name{regex}`
**Description:** Parameter must match the specified regular expression

**Examples:**
- `/:id{[0-9]+}` - Numeric IDs only
- `/:slug{[a-z-]+}` - Lowercase letters and hyphens only
- `/:filename{.+\\.png}` - Filenames ending with `.png`

### Wildcard Patterns

#### Single Segment Wildcard

**Pattern:** `/*`
**Description:** Matches any single URL segment

**Example:** `/static/*` matches `/static/css`, `/static/js` but not `/static/css/style.css`

#### Multi-Segment Wildcard

**Pattern:** `/**` or `/*path`
**Description:** Matches multiple URL segments including slashes

### Pattern Matching Rules

1. **Exact matches take precedence** over parameter matches
2. **Registration order matters** - first registered route wins
3. **Parameter names must be valid JavaScript identifiers**
4. **Regex patterns are evaluated as written** (no automatic anchoring)

## Route Organization

### Method Chaining

Routes can be chained for the same path pattern.

**Signature:**
```ts
app.get(path, handler).post(handler).delete(handler)
```

**Usage:** Subsequent chained methods apply to the same path as the first method.

### Route Grouping

#### Sub-Applications with `.route()`

Mount sub-applications at specific base paths.

**Signature:**
```ts
.route(basePath: string, subApp: Hono): Hono
```

**Parameters:**
- `basePath`: `string` - Base path for mounting the sub-application
- `subApp`: `Hono` - Sub-application instance

**Returns:** `Hono` - The parent application instance

**Route Resolution:** Routes from `subApp` are prefixed with `basePath`

#### Base Path Setting

Set a base path for all routes in an application.

**Signature:**
```ts
.basePath(path: string): Hono
```

**Parameters:**
- `path`: `string` - Base path to prepend to all routes

**Returns:** `Hono` - New application instance with base path applied

**Note:** Creates a new instance; does not modify the original.

## Advanced Routing

### Custom Path Extraction

Override the default URL path extraction for advanced routing scenarios.

**Configuration:**
```ts
new Hono({
  getPath: (request: Request) => string
})
```

**Use Cases:**
- Hostname-based routing
- Header-based routing  
- Custom URL parsing logic

### Hostname Routing

Route based on the request hostname by including it in the path pattern.

**Requirements:**
- Custom `getPath` function that includes hostname
- Route patterns that include the hostname

### Header-Based Routing

Route based on request headers (e.g., `Host`, `User-Agent`).

**Implementation:** Use custom `getPath` function to incorporate header values into the routing path.

## Route Execution Rules

### Priority and Ordering

1. **Registration order determines precedence**
2. **First matching route is executed**
3. **Route execution stops after handler completes**
4. **Middleware executes before handlers**

### Middleware Integration

- **Global middleware:** Applied to all routes (registered with `.use()`)
- **Path-specific middleware:** Applied to routes matching specific patterns
- **Route-level middleware:** Passed as additional handlers to route methods

### Execution Flow

1. Global middleware (in registration order)
2. Path-specific middleware (in registration order)  
3. Route handler
4. Response processing

## Strict Mode

Controls whether trailing slashes are significant in route matching.

**Default:** `true` (strict mode enabled)

**Strict Mode (`true`):**
- `/path` and `/path/` are different routes
- Exact path matching required

**Non-Strict Mode (`false`):**
- `/path` and `/path/` match the same route
- Trailing slash is ignored

**Configuration:**
```ts
new Hono({ strict: false })
```

## Error Cases

### Route Registration Errors

- **Invalid path patterns:** Malformed regex in parameter constraints
- **Duplicate exact patterns:** Later registration overwrites earlier one
- **Invalid method names:** Non-string HTTP methods

### Runtime Errors

- **No matching route:** Results in 404 Not Found (unless custom handler set)
- **Handler errors:** Caught by error handling middleware or global error handler

### Common Pitfalls

1. **Grouping order:** Sub-applications must be configured before mounting
2. **Wildcard precedence:** Overly broad patterns can shadow specific routes
3. **Parameter access:** Parameters only available in routes with parameter patterns

## Performance Considerations

- **Route complexity:** Regex patterns are slower than simple parameter matching
- **Route count:** Large numbers of routes may impact lookup performance
- **Router selection:** Different router implementations have different performance characteristics

## See Also

- [Hono Application Specification](/docs/api/hono) - Route registration methods
- [Context Specification](/docs/api/context) - Handler context and parameters
- [HonoRequest Specification](/docs/api/request) - Parameter access methods
- [Routing Examples](/docs/api/routing-examples) - Practical usage patterns
