# Exception Specification

The HTTPException class provides a standardized way to handle HTTP errors in Hono applications with proper status codes and custom responses.

## Type Definition

```ts
class HTTPException extends Error {
  readonly res?: Response
  readonly status: number
}
```

**Properties:**
- `status`: HTTP status code for the error
- `res`: Optional custom Response object
- `message`: Error message (inherited from Error)
- `cause`: Optional error cause (inherited from Error)

**Import:** `import { HTTPException } from 'hono/http-exception'`

## Constructor

### `new HTTPException(status, options?)`

Creates a new HTTPException instance.

**Parameters:**
- `status`: `number` - HTTP status code (400-599)
- `options` (optional): `HTTPExceptionOptions` - Exception configuration

**Options Type:**
```ts
type HTTPExceptionOptions = {
  res?: Response        // Custom response object
  message?: string      // Error message  
  cause?: unknown       // Error cause
}
```

**Returns:** `HTTPException` instance

## Constructor Overloads

### Basic Status Code

```ts
new HTTPException(404)
```

Creates exception with status code only.

### Status Code with Message

```ts
new HTTPException(400, { message: 'Invalid request data' })
```

Creates exception with custom message.

### Status Code with Custom Response

```ts
new HTTPException(401, { res: customResponse })
```

Creates exception with pre-built Response object.

### Status Code with Cause

```ts
new HTTPException(500, { message: 'Database error', cause: dbError })
```

Creates exception wrapping another error.

## Instance Methods

### `getResponse(): Response`

Generate or return the HTTP response for this exception.

**Returns:** `Response` - HTTP response object

**Behavior:**
- If `res` property exists, returns the custom response
- Otherwise, generates default response with status and message
- Default response uses `text/plain` content type

**Default Response Format:**
```ts
new Response(this.message, { status: this.status })
```

## Error Handling Integration

### Global Error Handler

HTTPException integrates with Hono's global error handler:

```ts
app.onError((err, c) => {
  if (err instanceof HTTPException) {
    return err.getResponse()
  }
  // Handle other error types
})
```

**Processing Flow:**
1. Exception thrown in handler or middleware
2. Bubbles up to global error handler
3. Error handler checks `instanceof HTTPException`
4. Calls `getResponse()` to get HTTP response
5. Response sent to client

### Middleware Error Handling

HTTPException can be thrown from middleware:

```ts
app.use('/protected/*', async (c, next) => {
  if (!isAuthenticated(c)) {
    throw new HTTPException(401, { message: 'Authentication required' })
  }
  await next()
})
```

**Behavior:**
- Stops middleware chain execution
- Bubbles up to error handler
- Bypasses subsequent middleware and route handlers

## Status Code Guidelines

### Client Errors (4xx)

- **400 Bad Request**: Malformed or invalid request data
- **401 Unauthorized**: Authentication required or failed
- **403 Forbidden**: Access denied (authenticated but not authorized)
- **404 Not Found**: Resource does not exist
- **405 Method Not Allowed**: HTTP method not supported for resource
- **409 Conflict**: Resource conflict (e.g., duplicate creation)
- **422 Unprocessable Entity**: Request format valid but semantically incorrect
- **429 Too Many Requests**: Rate limit exceeded

### Server Errors (5xx)

- **500 Internal Server Error**: Unexpected server error
- **502 Bad Gateway**: Upstream service error
- **503 Service Unavailable**: Server temporarily overloaded or down
- **504 Gateway Timeout**: Upstream service timeout

## Custom Response Patterns

### JSON Error Response

```ts
const jsonResponse = new Response(
  JSON.stringify({
    error: 'Validation failed',
    code: 'INVALID_INPUT',
    details: validationErrors
  }),
  {
    status: 400,
    headers: { 'Content-Type': 'application/json' }
  }
)

throw new HTTPException(400, { res: jsonResponse })
```

### Response with Custom Headers

```ts
const response = new Response('Rate limit exceeded', {
  status: 429,
  headers: {
    'Retry-After': '60',
    'X-RateLimit-Limit': '100',
    'X-RateLimit-Remaining': '0',
    'X-RateLimit-Reset': '1640995200'
  }
})

throw new HTTPException(429, { res: response })
```

### Redirect Response

```ts
const redirectResponse = new Response(null, {
  status: 302,
  headers: { 'Location': '/login' }
})

throw new HTTPException(302, { res: redirectResponse })
```

## Error Cause Usage

### Wrapping Lower-Level Errors

The `cause` property preserves original error information:

```ts
try {
  await databaseOperation()
} catch (dbError) {
  throw new HTTPException(500, {
    message: 'Database operation failed',
    cause: dbError
  })
}
```

**Benefits:**
- Preserves stack traces for debugging
- Allows error analysis without exposing internals
- Maintains error context through exception chain

### Error Chain Access

```ts
app.onError((err, c) => {
  if (err instanceof HTTPException && err.cause) {
    console.error('Original error:', err.cause)
    console.error('HTTP error:', err.message)
  }
  return err.getResponse()
})
```

## Validation Integration

### Form Validation Errors

```ts
const errors = validateForm(formData)
if (errors.length > 0) {
  const errorResponse = new Response(
    JSON.stringify({ 
      error: 'Validation failed', 
      fields: errors 
    }),
    { 
      status: 422,
      headers: { 'Content-Type': 'application/json' }
    }
  )
  throw new HTTPException(422, { res: errorResponse })
}
```

### Schema Validation

```ts
try {
  const validData = schema.parse(requestData)
} catch (validationError) {
  throw new HTTPException(400, {
    message: 'Invalid request format',
    cause: validationError
  })
}
```

## Performance Considerations

### Exception Creation Cost

- HTTPException creation is lightweight
- Custom Response creation may have overhead
- Consider reusing Response objects for common errors

### Error Handler Performance

- Use `instanceof HTTPException` check early
- Avoid complex processing in error handlers
- Consider error response caching for common cases

## Best Practices

### Error Message Guidelines

- **Client errors**: Provide helpful, actionable messages
- **Server errors**: Use generic messages to avoid information leakage
- **Development**: Include detailed error information
- **Production**: Sanitize error messages for security

### Response Consistency

Maintain consistent error response format:

```ts
interface ErrorResponse {
  error: string
  message?: string
  code?: string
  timestamp?: string
  requestId?: string
}
```

### Error Logging

```ts
app.onError((err, c) => {
  // Log error details
  console.error({
    error: err.message,
    status: err instanceof HTTPException ? err.status : 500,
    path: c.req.path,
    method: c.req.method,
    cause: err.cause,
    timestamp: new Date().toISOString()
  })
  
  return err instanceof HTTPException 
    ? err.getResponse() 
    : c.json({ error: 'Internal Server Error' }, 500)
})
```

## See Also

- [Exception Examples](/docs/api/exception-examples) - Practical error handling patterns
- [Hono Specification](/docs/api/hono) - Global error handler configuration  
- [Context Specification](/docs/api/context) - Response generation methods
- [Middleware Guide](/docs/guides/middleware) - Middleware error handling
