# Hono Specification

The `Hono` class is the main application class that handles routing, middleware, and request-response processing.

## Type Definition

```ts
class Hono<Env = {}, Schema = {}, BasePath extends string = '/'>
```

**Type Parameters:**
- `Env`: Environment type including `Bindings` and `Variables`
- `Schema`: API schema type for type-safe routing
- `BasePath`: Base path string literal type

## Constructor

### `new Hono(options?)`

Creates a new Hono application instance.

**Parameters:**
- `options` (optional): `HonoOptions<Env>` - Configuration options

**Options:**
- `strict?: boolean` - Enable strict trailing slash handling (default: `true`)
- `router?: Router` - Custom router instance (default: `SmartRouter`)
- `getPath?: (request: Request) => string` - Custom path extraction function

**Returns:** `Hono<Env, Schema, BasePath>`

## Core Methods

### HTTP Method Routing

#### `get(path, ...handlers)`
#### `post(path, ...handlers)`
#### `put(path, ...handlers)`
#### `delete(path, ...handlers)`
#### `head(path, ...handlers)`
#### `options(path, ...handlers)`
#### `patch(path, ...handlers)`

Register a handler for the specified HTTP method and path.

**Parameters:**
- `path`: `string` - Route path pattern
- `...handlers`: `Handler[]` - One or more request handlers or middleware

**Returns:** `Hono` - The application instance (for method chaining)

**Notes:**
- Path patterns support parameters, wildcards, and regex constraints
- Multiple handlers are executed in order, use `await next()` in middleware

#### `all(path, ...handlers)`

Register a handler that matches all HTTP methods.

**Parameters:**
- `path`: `string` - Route path pattern  
- `...handlers`: `Handler[]` - One or more request handlers or middleware

**Returns:** `Hono` - The application instance (for method chaining)

#### `on(method, path, ...handlers)`

Register a handler for custom HTTP methods or multiple methods/paths.

**Parameters:**
- `method`: `string | string[]` - HTTP method(s)
- `path`: `string | string[]` - Route path pattern(s)
- `...handlers`: `Handler[]` - One or more request handlers or middleware

**Returns:** `Hono` - The application instance (for method chaining)

### Middleware Registration

#### `use(path?, ...middleware)`

Register middleware that executes for all matching routes.

**Parameters:**
- `path` (optional): `string` - Path pattern to match (default: `'*'`)
- `...middleware`: `MiddlewareHandler[]` - Middleware functions

**Returns:** `Hono` - The application instance (for method chaining)

**Notes:**
- Middleware executes in registration order
- Must call `await next()` to continue to subsequent middleware/handlers

### Application Composition

#### `route(path, app)`

Mount a sub-application at the specified path.

**Parameters:**
- `path`: `string` - Base path for the sub-application
- `app`: `Hono` - Hono application instance to mount

**Returns:** `Hono` - The application instance (for method chaining)

**Notes:**
- Routes from the sub-application are prefixed with the mount path
- Sub-application middleware and error handlers are preserved

#### `basePath(path)`

Set a base path for all routes in this application.

**Parameters:**
- `path`: `string` - Base path string

**Returns:** `Hono` - A new Hono instance with the base path applied

**Notes:**
- Returns a new instance, does not modify the original
- All routes registered on the returned instance will be prefixed

#### `mount(path, fetchLike)`

Mount a fetch-compatible application or function.

**Parameters:**
- `path`: `string` - Base path for mounting
- `fetchLike`: `(request: Request, ...args: any[]) => Response | Promise<Response>` - Fetch-compatible handler

**Returns:** `Hono` - The application instance (for method chaining)

**Notes:**
- Useful for integrating with other frameworks or external applications
- The mounted application receives the full Request object

## Error and Response Handling

### `notFound(handler)`

Register a custom handler for 404 Not Found responses.

**Parameters:**
- `handler`: `NotFoundHandler` - Handler function for 404 responses

**Type Definition:**
```ts
type NotFoundHandler = (c: Context) => Response | Promise<Response>
```

**Returns:** `Hono` - The application instance (for method chaining)

**Notes:**
- Executes when no routes match the request
- Only one notFound handler can be registered (last one wins)

### `onError(handler)`

Register a global error handler for uncaught exceptions.

**Parameters:**
- `handler`: `ErrorHandler` - Error handling function

**Type Definition:**
```ts
type ErrorHandler = (err: Error, c: Context) => Response | Promise<Response>
```

**Returns:** `Hono` - The application instance (for method chaining)

**Notes:**
- Catches errors thrown by handlers or middleware
- Only one error handler can be registered (last one wins)
- Should return a Response object

## Application Entry Points

### `fetch(request, env?, executionCtx?)`

Main entry point for handling HTTP requests.

**Parameters:**
- `request`: `Request` - The HTTP request object
- `env` (optional): `Env['Bindings']` - Environment bindings (e.g., KV stores, secrets)
- `executionCtx` (optional): `ExecutionContext` - Execution context (Cloudflare Workers)

**Returns:** `Response | Promise<Response>`

**Notes:**
- This is the primary method called by the runtime
- Compatible with Web Standards fetch API
- Handles routing, middleware execution, and response generation

### `request(path, options?)`

Create a test request to the application.

**Parameters:**
- `path`: `string | Request` - URL path or complete Request object
- `options` (optional): `RequestInit` - Request options (method, headers, body, etc.)

**Returns:** `Promise<Response>`

**Notes:**
- Primarily used for testing
- If first parameter is a string, creates a GET request by default
- If first parameter is a Request object, options parameter is ignored

### `fire()` *(Deprecated)*

**Status:** Deprecated - Use `fire()` from `hono/service-worker` instead

Automatically adds a global `fetch` event listener.

**Returns:** `void`

**Notes:**
- Only useful for Service Worker environments
- Executes `addEventListener('fetch', ...)` automatically
- Use `hono/service-worker` for new projects

## Configuration Options

### Strict Mode

When `strict: true` (default):
- `/hello` and `/hello/` are treated as different routes
- Trailing slashes must match exactly

When `strict: false`:
- `/hello` and `/hello/` are treated as the same route
- More flexible path matching

### Custom Router

The `router` option allows specifying which routing algorithm to use:
- `SmartRouter` (default) - Automatically selects best router
- `RegExpRouter` - Fastest for most cases
- `TrieRouter` - Supports all patterns
- `LinearRouter` - Simple linear matching
- `PatternRouter` - Smallest bundle size

### Custom Path Extraction

The `getPath` function allows custom URL parsing logic:

**Type:**
```ts
getPath?: (request: Request) => string
```

**Use Cases:**
- Hostname-based routing
- Header-based routing (e.g., User-Agent)
- Custom URL parsing logic

## Type Safety

### Environment Types

```ts
type HonoEnv = {
  Bindings: {
    // Environment variables, KV stores, etc.
    API_KEY: string
    DB: KVNamespace
  }
  Variables: {
    // Per-request variables set via c.set()
    user: User
    requestId: string
  }
}

const app = new Hono<HonoEnv>()
```

### Schema Types

For API type safety with libraries like Hono RPC:

```ts
type AppSchema = {
  '/api/users': {
    $get: {
      query: { page?: string }
      response: { users: User[] }
    }
  }
}

const app = new Hono<Env, AppSchema>()
```

## Performance Characteristics

- **Cold start**: Optimized for serverless environments
- **Memory usage**: Minimal footprint
- **Request handling**: High throughput with proper router selection
- **Bundle size**: Varies by router choice (15KB - 40KB typical)

## See Also

- [Hono Examples](/docs/api/hono-examples) - Practical usage examples and patterns
- [Routing Specification](/docs/api/routing) - Complete routing system specification
- [Context Specification](/docs/api/context) - Request-response context object
- [Router Presets](/docs/api/presets) - Available router configurations