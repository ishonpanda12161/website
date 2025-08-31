# Hono Application Specification

The `Hono` class is the core application object for creating web applications and APIs.

## Type Definition

```ts
class Hono<
  Env extends Env = {},
  Schema extends Schema = {},
  BasePath extends string = '/'
>
```

**Type Parameters:**
- `Env`: Environment bindings type (e.g., Cloudflare Workers KV, secrets)
- `Schema`: API schema type for RPC and type safety
- `BasePath`: Base path string for the application

## Constructor

### `new Hono(options?)`

Creates a new Hono application instance.

**Parameters:**
- `options` (optional): `HonoOptions` - Configuration options

**HonoOptions:**
```ts
interface HonoOptions {
  strict?: boolean
  router?: Router
  getPath?: (request: Request) => string
}
```

- `strict`: Whether to distinguish `/path` and `/path/` (default: `true`)
- `router`: Custom router instance to use
- `getPath`: Custom path extraction function for advanced routing

**Returns:** `Hono` - New application instance

## Core Methods

### HTTP Method Routing

#### `.get(path?, ...handlers)`
#### `.post(path?, ...handlers)`
#### `.put(path?, ...handlers)`
#### `.delete(path?, ...handlers)`
#### `.patch(path?, ...handlers)`
#### `.head(path?, ...handlers)`
#### `.options(path?, ...handlers)`

Register handlers for specific HTTP methods.

**Parameters:**
- `path` (optional): `string | string[]` - Path pattern(s) to match
- `...handlers`: `Handler | Middleware[]` - Handler function(s) to execute

**Returns:** `Hono` - The application instance (for chaining)

**Path Patterns:**
- Static paths: `/users`, `/api/v1/posts`
- Parameters: `/users/:id`, `/posts/:slug`
- Optional parameters: `/api/animal/:type?`  
- Wildcards: `/static/*`, `/files/*/download`
- Regex: `/post/:date{[0-9]+}/:title{[a-z]+}`

### Generic Routing

#### `.all(path?, ...handlers)`

Register handlers for all HTTP methods.

**Parameters:**
- `path` (optional): `string | string[]` - Path pattern(s)
- `...handlers`: `Handler | Middleware[]` - Handler function(s)

**Returns:** `Hono` - The application instance

#### `.on(methods, paths, ...handlers)`

Register handlers for specific methods and paths.

**Parameters:**
- `methods`: `HTTPMethod | HTTPMethod[]` - HTTP method(s)
- `paths`: `string | string[]` - Path pattern(s)
- `...handlers`: `Handler | Middleware[]` - Handler function(s)

**Returns:** `Hono` - The application instance

**HTTPMethod:** `'GET' | 'POST' | 'PUT' | 'DELETE' | 'PATCH' | 'HEAD' | 'OPTIONS'` or custom methods

### Middleware

#### `.use(path?, ...middleware)`

Register middleware functions.

**Parameters:**
- `path` (optional): `string | string[]` - Path pattern(s) to apply middleware
- `...middleware`: `Middleware[]` - Middleware function(s)

**Returns:** `Hono` - The application instance

**Middleware Signature:**
```ts
type Middleware = (c: Context, next: Next) => Promise<Response | void>
type Next = () => Promise<void>
```

### Application Organization

#### `.route(path, app)`

Mount a sub-application at the specified path.

**Parameters:**
- `path`: `string` - Base path for the sub-application
- `app`: `Hono` - Sub-application instance

**Returns:** `Hono` - The application instance

#### `.basePath(path)`

Set a base path for the application.

**Parameters:**
- `path`: `string` - Base path string

**Returns:** `Hono` - New application instance with base path

#### `.mount(path, fetchFunction)`

Mount external applications or functions at the specified path.

**Parameters:**
- `path`: `string` - Mount path
- `fetchFunction`: `(request: Request, ...args: any[]) => Response | Promise<Response>` - Fetch-compatible function

**Returns:** `Hono` - The application instance

## Error Handling

#### `.notFound(handler)`

Set a custom 404 Not Found handler.

**Parameters:**
- `handler`: `NotFoundHandler` - Handler for 404 responses

**Handler Signature:**
```ts
type NotFoundHandler = (c: Context) => Response | Promise<Response>
```

**Returns:** `Hono` - The application instance

#### `.onError(handler)`

Set a global error handler.

**Parameters:**
- `handler`: `ErrorHandler` - Error handling function

**Handler Signature:**
```ts  
type ErrorHandler = (error: Error, c: Context) => Response | Promise<Response>
```

**Returns:** `Hono` - The application instance

## Runtime Integration

#### `.fetch(request, env?, executionCtx?)`

Main entry point for handling requests. Used by runtime platforms.

**Parameters:**
- `request`: `Request` - Incoming HTTP request
- `env` (optional): `Env` - Environment bindings
- `executionCtx` (optional): `ExecutionContext` - Execution context (Cloudflare Workers)

**Returns:** `Promise<Response>` - HTTP response

**Usage:** Export this method for runtime platforms like Cloudflare Workers, Bun, etc.

#### `.request(input, init?)`

Send a request to the application (primarily for testing).

**Parameters:**
- `input`: `string | Request` - URL string or Request object
- `init` (optional): `RequestInit` - Request options

**Returns:** `Promise<Response>` - Response from the application

**Note:** When passing a string, creates a GET request to that path.

#### `.fire()` *(deprecated)*

Automatically adds a global `fetch` event listener for Service Worker environments.

**Returns:** `void`

**Deprecation:** Use `fire()` from `hono/service-worker` instead.

## Configuration

### Strict Mode

Controls whether `/path` and `/path/` are treated as different routes.

**Default:** `true` (distinguishes paths)
**Disabled:** Both paths match the same route

### Router Selection

Specify which router implementation to use:

**Available Routers:**
- `SmartRouter` (default): Adaptive router for most use cases
- `RegExpRouter`: Regular expression-based routing
- `TrieRouter`: Tree-based routing for many static routes
- `LinearRouter`: Simple linear search (development/testing)
- `PatternRouter`: Minimal pattern-based routing

### Custom Path Extraction

Override how paths are extracted from requests for advanced routing scenarios like hostname-based routing.

**Example Use Cases:**
- Subdomain routing
- Host header routing  
- Custom URL parsing

## See Also

- [Context Specification](/docs/api/context) - Request/response handling
- [Routing Specification](/docs/api/routing) - Route patterns and organization
- [Exception Specification](/docs/api/exception) - Error handling
- [Hono Examples](/docs/api/hono-examples) - Practical usage examples
