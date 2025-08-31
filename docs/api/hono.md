# Hono Application Specification

The `Hono` class is the core application object for creating HTTP servers and handling requests.

## Type Definition

```ts
class Hono<E extends Env = Env, S extends Schema = {}, BasePath extends string = '/'>
```

**Type Parameters:**

- `E`: Environment bindings type (e.g., Cloudflare Workers bindings)
- `S`: Schema type for type-safe routing
- `BasePath`: Base path string for the application

## Constructor

### `new Hono(options?)`

Creates a new Hono application instance.

**Parameters:**

- `options` (optional): `HonoOptions`
  - `strict?: boolean` - Enable strict routing (default: `true`)
  - `router?: Router` - Custom router instance
  - `getPath?: (req: Request) => string` - Custom path extraction function

**Returns:** `Hono<E, S, BasePath>`

**Example:**

```ts
import { Hono } from 'hono'
import { RegExpRouter } from 'hono/router/reg-exp-router'

// Basic usage
const app = new Hono()

// With options
const strictApp = new Hono({
  strict: false,
  router: new RegExpRouter(),
})
```

## HTTP Method Routing

### `get(path, ...handlers)`

### `post(path, ...handlers)`

### `put(path, ...handlers)`

### `delete(path, ...handlers)`

### `head(path, ...handlers)`

### `options(path, ...handlers)`

### `patch(path, ...handlers)`

Register handlers for specific HTTP methods.

**Parameters:**

- `path`: `string` - Route path pattern
- `...handlers`: `Handler[]` - Request handlers and middleware

**Returns:** `Hono` - Returns the Hono instance for chaining

**See:** [Routing Specification](/docs/api/routing)

### `all(path, ...handlers)`

Register handlers for all HTTP methods.

**Parameters:**

- `path`: `string` - Route path pattern
- `...handlers`: `Handler[]` - Request handlers and middleware

**Returns:** `Hono`

### `on(method, path, ...handlers)`

Register handlers for custom or multiple HTTP methods.

**Parameters:**

- `method`: `string | string[]` - HTTP method(s)
- `path`: `string | string[]` - Route path pattern(s)
- `...handlers`: `Handler[]` - Request handlers and middleware

**Returns:** `Hono`

## Middleware

### `use(path?, ...middleware)`

Register middleware for request processing.

**Parameters:**

- `path` (optional): `string` - Path pattern to apply middleware
- `...middleware`: `MiddlewareHandler[]` - Middleware functions

**Returns:** `Hono`

**Notes:**

- Middleware executes in registration order
- If no path is specified, middleware applies to all routes

## Application Composition

### `route(path, app)`

Mount a sub-application at the specified path.

**Parameters:**

- `path`: `string` - Mount path
- `app`: `Hono` - Sub-application to mount

**Returns:** `Hono`

**Notes:**

- Routes from the sub-application are prefixed with the mount path
- Middleware from the sub-application is preserved

### `basePath(path)`

Set a base path for all routes in the application.

**Parameters:**

- `path`: `string` - Base path to prepend to all routes

**Returns:** `Hono<E, S, BasePath>`

**Notes:**

- All subsequent routes will be prefixed with this path
- Returns a new typed instance with the base path

### `mount(path, fetchFunction)`

Mount an external application or framework.

**Parameters:**

- `path`: `string` - Mount path
- `fetchFunction`: `(request: Request, ...args: any[]) => Response | Promise<Response>` - External app's fetch function

**Returns:** `Hono`

**Notes:**

- Used for integrating other frameworks or applications
- The mounted application receives requests with the mount path stripped

## Error Handling

### `notFound(handler)`

Set a custom handler for 404 Not Found responses.

**Parameters:**

- `handler`: `NotFoundHandler<E>` - Function to handle 404 responses

**Returns:** `Hono`

**Type:**

```ts
type NotFoundHandler<E extends Env = Env> = (
  c: Context<E>
) => Response | Promise<Response>
```

### `onError(handler)`

Set a global error handler for uncaught exceptions.

**Parameters:**

- `handler`: `ErrorHandler<E>` - Function to handle errors

**Returns:** `Hono`

**Type:**

```ts
type ErrorHandler<E extends Env = Env> = (
  error: Error,
  c: Context<E>
) => Response | Promise<Response>
```

## Request Handling

### `fetch(request, env?, executionContext?)`

The main entry point for handling HTTP requests.

**Parameters:**

- `request`: `Request` - The incoming HTTP request
- `env` (optional): `E` - Environment bindings (platform-specific)
- `executionContext` (optional): `ExecutionContext` - Execution context (platform-specific)

**Returns:** `Promise<Response>`

**Notes:**

- This is the primary method called by the runtime environment
- Environment and execution context are platform-specific

### `request(input, init?)`

Send a request to the application for testing purposes.

**Parameters:**

- `input`: `string | URL | Request` - Request URL or Request object
- `init` (optional): `RequestInit` - Request options

**Returns:** `Promise<Response>`

**Notes:**

- Primarily used for unit testing
- Creates a mock environment for the request

## Legacy Methods

### `fire()`

**Status:** Deprecated - Use `fire()` from `hono/service-worker` instead

Automatically adds a global fetch event listener for Service Worker environments.

**Returns:** `void`

**Notes:**

- Only for Service Worker API environments
- Deprecated in favor of explicit Service Worker integration

## Configuration Options

### Strict Mode

**Default:** `true`

When enabled, trailing slashes in routes are significant:

- `/hello` and `/hello/` are treated as different routes

When disabled, trailing slashes are ignored:

- `/hello` and `/hello/` are treated as the same route

### Custom Router

Specify which router implementation to use:

**Available Routers:**

- `SmartRouter` (default) - Automatically selects optimal router
- `RegExpRouter` - Uses regular expressions for matching
- `TrieRouter` - Uses trie data structure for matching
- `LinearRouter` - Linear search through routes
- `PatternRouter` - Simple pattern matching

### Custom Path Extraction

Override how paths are extracted from requests for advanced routing scenarios like hostname-based routing.

**Type:**

```ts
type GetPath = (req: Request) => string
```

## Type System Integration

### Environment Bindings

Type the environment bindings for your platform:

```ts
type Bindings = {
  DB: D1Database // Cloudflare D1
  KV: KVNamespace // Cloudflare KV
  API_KEY: string // Environment variable
}

const app = new Hono<{ Bindings: Bindings }>()
```

### Context Variables

Type the variables that can be stored in the request context:

```ts
type Variables = {
  user: User
  requestId: string
  startTime: number
}

const app = new Hono<{ Variables: Variables }>()
```

### Combined Type Usage

```ts
const app = new Hono<{
  Bindings: Bindings
  Variables: Variables
}>()

app.use('*', async (c, next) => {
  const apiKey = c.env.API_KEY // Typed as string
  c.set('requestId', 'req-123') // Typed variable
  await next()
})
```

## Method Chaining

All routing methods return the Hono instance, enabling method chaining:

```ts
app
  .get('/users', handler)
  .post('/users', handler)
  .put('/users/:id', handler)
  .delete('/users/:id', handler)
```

## Application Lifecycle

1. **Construction** - Create Hono instance with optional configuration
2. **Route Registration** - Register routes, middleware, and handlers
3. **Application Mounting** - Mount sub-applications if needed
4. **Error Handler Setup** - Configure global error handling
5. **Request Processing** - Handle incoming requests via `fetch()`

## Integration Patterns

### Platform Entry Points

Different platforms require different entry point patterns:

**Cloudflare Workers:**

```ts
export default app
// or
export default { fetch: app.fetch }
```

**Bun:**

```ts
export default {
  port: 3000,
  fetch: app.fetch,
}
```

**Node.js with serve():**

```ts
import { serve } from 'bun'

serve({
  port: 3000,
  fetch: app.fetch,
})
```

## See Also

- [Context Specification](/docs/api/context) - Request-response context object
- [Routing Specification](/docs/api/routing) - Route registration and matching
- [Exception Specification](/docs/api/exception) - Error handling patterns
- [Hono Examples](/docs/api/hono-examples) - Practical usage patterns
