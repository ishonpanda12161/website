# Exception Specification

Hono provides structured error handling through the `HTTPException` class and global error handling mechanisms for managing HTTP errors and custom error responses.

## HTTPException Class

### Constructor

```ts
new HTTPException(status, options?)
```

Parameters:
- status: `number` — HTTP status code (e.g., 401, 404, 500)
- options (optional): `HTTPExceptionOptions`
  - message: `string` — Custom error message
  - res: `Response` — Custom Response object to return
  - cause: `unknown` — Underlying cause of the error

Returns: `HTTPException` instance

### Methods

#### getResponse()

Generate the HTTP response for the exception.

```ts
getResponse(): Response
```

Returns: `Response` — HTTP response based on status and options

**Behavior:**
- If `options.res` was provided in constructor, returns that Response
- Otherwise, creates a Response with the status code and message

## Throwing HTTPExceptions

### Basic Usage

Throw an HTTPException to immediately halt request processing and return an error response:

```ts
import { HTTPException } from 'hono/http-exception'

// Simple error with status and message
throw new HTTPException(401, { message: 'Authentication required' })

// Error with custom response
const errorResponse = new Response('Unauthorized', {
  status: 401,
  headers: { 'WWW-Authenticate': 'Bearer' }
})
throw new HTTPException(401, { res: errorResponse })
```

### With Cause Chain

Preserve the original error context using the `cause` option:

```ts
app.post('/protected', async (c, next) => {
  try {
    await validateToken(c.req.header('authorization'))
  } catch (originalError) {
    throw new HTTPException(401, { 
      message: 'Token validation failed',
      cause: originalError 
    })
  }
  await next()
})
```

## Global Error Handling

### app.onError()

Register a global error handler for all unhandled exceptions:

```ts
app.onError(handler)
```

Parameters:
- handler: `(err: Error, c: Context) => Response | Promise<Response>`

**Handler receives:**
- err: `Error` — The thrown error (may be HTTPException or any Error)
- c: `Context` — Request context for creating responses

### Error Handler Patterns

```ts
app.onError((err, c) => {
  // Handle HTTPException specifically
  if (err instanceof HTTPException) {
    return err.getResponse()
  }
  
  // Handle other error types
  console.error('Unexpected error:', err)
  return c.text('Internal Server Error', 500)
})
```

## Error Response Flow

1. **Exception thrown** — HTTPException thrown from middleware/handler
2. **Processing halted** — Request processing stops immediately  
3. **Error handler invoked** — Global `onError` handler receives the exception
4. **Response generated** — Handler returns appropriate Response object

## Integration with Middleware

HTTPExceptions work seamlessly with middleware patterns:

```ts
// Authentication middleware
app.use('/protected/*', async (c, next) => {
  const token = c.req.header('authorization')
  
  if (!token) {
    throw new HTTPException(401, { message: 'Missing authorization header' })
  }
  
  if (!await isValidToken(token)) {
    throw new HTTPException(403, { message: 'Invalid token' })
  }
  
  await next()
})

// Routes are only reached if authentication passes
app.get('/protected/resource', (c) => {
  return c.json({ data: 'protected content' })
})
```

## Status Code Conventions

Common HTTP status codes used with HTTPException:

- `400` — Bad Request (invalid client input)
- `401` — Unauthorized (authentication required)
- `403` — Forbidden (access denied)
- `404` — Not Found (resource doesn't exist)
- `422` — Unprocessable Entity (validation failure)
- `500` — Internal Server Error (server-side errors)

## See Also

- [Context](/docs/api/context) — Request context and response helpers
- [Hono Object](/docs/api/hono) — Global error handler registration
- [Middleware](/docs/guides/middleware) — Using exceptions in middleware patterns
