# Context Specification

The `Context` object is instantiated for each HTTP request and contains methods and properties for handling the current request-response lifecycle.

## Type Definition

```ts
Context<Env = {}, Variables = {}>
```

**Type Parameters:**
- `Env`: Type for environment bindings (e.g., Cloudflare Workers secrets, KV namespaces)
- `Variables`: Type for per-request variables set via `c.set()`

## Core Properties

### `req`

The enhanced request object for the current request.

**Type:** [`HonoRequest`](/docs/api/request)

**Example:**
```ts
// Access path parameters
const id = c.req.param('id')

// Access query string
const query = c.req.query('q')

// Parse request body
const body = await c.req.parseBody()
```

**See also:** [HonoRequest](/docs/api/request)

### `res`

The underlying response object.

**Type:** `Response`

**Notes:**
- Generally not used directly; use the helper methods instead
- Use when you need direct access to the Response object

### `env`

Access to environment bindings (e.g., Cloudflare Workers KV, secrets).

**Type:** `Env` (from generic parameter)

**Example:**
```ts
// Access a KV namespace
const value = await c.env.MY_KV.get('key')

// Access a secret
const apiKey = c.env.API_KEY
```

### `error`

If set, contains an error that occurred during processing.

**Type:** `Error | undefined`

**Notes:**
- Set automatically by Hono when errors are caught
- Can be checked in error handling middleware

## Response Helper Methods

All response helper methods return either a `Response` or a `Promise<Response>`.

### `text(body, status?, headers?)`

Send a plain text response.

**Parameters:**
- `body`: `string` - Text content
- `status` (optional): `number` - HTTP status code (default: 200)
- `headers` (optional): `Record<string, string> | Headers` - HTTP headers

**Returns:** `Response`

**Example:**
```ts
return c.text('Hello world', 200, { 'X-Custom-Header': 'value' })
```

### `json(body, status?, headers?)`

Send a JSON response.

**Parameters:**
- `body`: `any` - Object to serialize as JSON
- `status` (optional): `number` - HTTP status code (default: 200)
- `headers` (optional): `Record<string, string> | Headers` - HTTP headers

**Returns:** `Response`

**Notes:**
- Automatically sets `Content-Type: application/json`
- Automatically serializes the body using `JSON.stringify()`

**Example:**
```ts
return c.json({ message: 'Success', data: { id: 123 } })
```

### `html(body, status?, headers?)`

Send an HTML response.

**Parameters:**
- `body`: `string` - HTML content
- `status` (optional): `number` - HTTP status code (default: 200)
- `headers` (optional): `Record<string, string> | Headers` - HTTP headers

**Returns:** `Response`

**Notes:**
- Automatically sets `Content-Type: text/html; charset=UTF-8`

**Example:**
```ts
return c.html('<h1>Hello world</h1>')
```

### `body(body, status?, headers?)`

Send a custom response body.

**Parameters:**
- `body`: `string | ReadableStream | ArrayBuffer | ArrayBufferView | Blob | null` - Response body
- `status` (optional): `number` - HTTP status code (default: 200)
- `headers` (optional): `Record<string, string> | Headers` - HTTP headers

**Returns:** `Response`

**Notes:**
- Use when other helper methods don't fit your needs
- More direct access to the Response constructor

### `notFound()`

Send a 404 Not Found response.

**Returns:** `Response`

**Notes:**
- Can be customized application-wide using `app.notFound()`

### `redirect(url, status?)`

Redirect to another URL.

**Parameters:**
- `url`: `string` - URL to redirect to
- `status` (optional): `number` - HTTP status code (default: 302)

**Returns:** `Response`

**Example:**
```ts
return c.redirect('/login')
return c.redirect('/permanent', 301) // Permanent redirect
```

## Headers and Status

### `status(code)`

Set the HTTP status code for the response.

**Parameters:**
- `code`: `number` - HTTP status code

**Returns:** `Context` - The context instance (for chaining)

**Example:**
```ts
return c.status(201).json({ created: true })
```

### `header(name, value)`

Set a response header.

**Parameters:**
- `name`: `string` - Header name
- `value`: `string` - Header value

**Returns:** `Context` - The context instance (for chaining)

**Example:**
```ts
return c.header('X-Custom', 'value').text('Response with custom header')
```

## Per-Request Variables

### `set(key, value)`

Set a per-request variable.

**Parameters:**
- `key`: `keyof Variables` - Variable name
- `value`: `Variables[keyof Variables]` - Variable value

**Returns:** `Context` - The context instance (for chaining)

**Notes:**
- Values are retained only within the same request
- Type-safe when using generics

**Example:**
```ts
c.set('user', user)
```

### `get(key)`

Get a per-request variable.

**Parameters:**
- `key`: `keyof Variables` - Variable name

**Returns:** `Variables[keyof Variables] | undefined`

**Example:**
```ts
const user = c.get('user')
```

### `var`

Access variables as properties (type-safe).

**Type:** `Variables`

**Example:**
```ts
const user = c.var.user
```

## Rendering

### `setRenderer(rendererFn)`

Set a custom renderer for the response.

**Parameters:**
- `rendererFn`: `ContextRenderer` - Renderer function

**Returns:** `void`

**Notes:**
- Often used in middleware to set up layout templates

### `render(content, args?)`

Render using the renderer set by `setRenderer`.

**Parameters:**
- `content`: `any` - Content to render
- `args` (optional): `any` - Additional arguments for the renderer

**Returns:** `Response | Promise<Response>`

**Notes:**
- Must call `setRenderer` first
- Allows for consistent template/layout usage

## Platform-Specific Features

### `executionCtx`

Access to Cloudflare's `ExecutionContext`.

**Type:** `ExecutionContext`

**Notes:**
- Cloudflare Workers specific
- Used for operations like `waitUntil()`

### `event`

Access to Cloudflare's `FetchEvent`.

**Type:** `FetchEvent`

**Notes:**
- **Deprecated** - Use `executionCtx` instead
- Only available when using Service Worker syntax

## Error Handling

If an error occurs during request processing, it will be set on `c.error` and can be handled in middleware or in the global error handler.

## Type Extension

### `ContextVariableMap`

To extend the type definitions for variables when using specific middleware:

```ts
declare module 'hono' {
  interface ContextVariableMap {
    customVar: string
  }
}
```

This allows for type-safe access to variables even when they're set by middleware.

## See Also

- [HonoRequest](/docs/api/request) - The enhanced request object
- [Routing](/docs/api/routing) - How routing works in Hono
- [Exception](/docs/api/exception) - Error handling in Hono
