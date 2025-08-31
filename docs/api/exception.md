# Exception Handling Specification

The HTTPException class provides structured HTTP error handling for Hono applications, allowing for consistent error responses with proper status codes and messages.

## HTTPException Class

### Type Definition

```ts
class HTTPException extends Error {
  readonly status: number
  readonly res?: Response
  readonly cause?: unknown
}
```

**Properties:**
- `status`: HTTP status code
- `res`: Optional custom Response object
- `cause`: Optional underlying error cause
- `message`: Error message (inherited from Error)

## Constructor

### `new HTTPException(status, options?)`

Create a new HTTP exception.

**Signature:**
```ts
new HTTPException(status: number, options?: HTTPExceptionOptions)
```

**Parameters:**
- `status`: `number` - HTTP status code (e.g., 400, 401, 403, 404, 500)
- `options` (optional): `HTTPExceptionOptions` - Additional configuration

**HTTPExceptionOptions:**
```ts
interface HTTPExceptionOptions {
  message?: string
  res?: Response
  cause?: unknown
}
```

- `message`: Custom error message
- `res`: Pre-built Response object to return
- `cause`: Underlying error or cause information

**Examples:**
```ts
// Basic exception with status
throw new HTTPException(401)

// With custom message
throw new HTTPException(403, { message: 'Access denied' })

// With custom response
throw new HTTPException(401, { res: unauthorizedResponse })

// With cause information
throw new HTTPException(500, { message: 'Database error', cause: dbError })
```

## Instance Methods

### `.getResponse()`

Get the HTTP response for this exception.

**Signature:**
```ts
getResponse(): Response
```

**Returns:** `Response` - HTTP response object

**Behavior:**
- If `res` option was provided in constructor, returns that Response
- Otherwise, creates a new Response with the error message and status code
- Default message is the HTTP status text for the status code

## Usage Patterns

### In Middleware

HTTPException is commonly thrown in middleware for authentication, authorization, and validation:

```ts
// Authentication middleware
app.use(async (c, next) => {
  const token = c.req.header('Authorization')
  if (!token) {
    throw new HTTPException(401, { message: 'Authorization required' })
  }
  await next()
})
```

### In Route Handlers

Thrown when request processing encounters errors:

```ts
app.get('/user/:id', async (c) => {
  const id = c.req.param('id')
  const user = await getUserById(id)
  
  if (!user) {
    throw new HTTPException(404, { message: 'User not found' })
  }
  
  return c.json(user)
})
```

### With Custom Response

For complete control over the error response:

```ts
const customErrorResponse = new Response(
  JSON.stringify({ error: 'Unauthorized', code: 'AUTH001' }),
  {
    status: 401,
    headers: {
      'Content-Type': 'application/json',
      'WWW-Authenticate': 'Bearer realm="api"'
    }
  }
)

throw new HTTPException(401, { res: customErrorResponse })
```

## Error Handling Integration

### Global Error Handler

HTTPException integrates with Hono's global error handling:

```ts
app.onError((err, c) => {
  if (err instanceof HTTPException) {
    return err.getResponse()
  }
  
  // Handle other errors
  console.error(err)
  return c.text('Internal Server Error', 500)
})
```

### Error Handler Signature

```ts
type ErrorHandler = (error: Error, c: Context) => Response | Promise<Response>
```

**Parameters:**
- `error`: The caught error (may be HTTPException or other Error types)
- `c`: Context object for the current request

**Returns:** Response object for the error

## Error Cause Tracking

### Using the `cause` Option

The `cause` option allows chaining errors for better debugging:

```ts
app.post('/process', async (c) => {
  try {
    await processData(c.req.json())
  } catch (originalError) {
    throw new HTTPException(500, { 
      message: 'Processing failed',
      cause: originalError 
    })
  }
})
```

### Cause Information Access

Access the original error through the `cause` property:

```ts
app.onError((err, c) => {
  if (err instanceof HTTPException && err.cause) {
    console.error('Original error:', err.cause)
  }
  
  return err instanceof HTTPException 
    ? err.getResponse()
    : c.text('Internal Error', 500)
})
```

## Status Code Guidelines

### Client Error Status Codes (4xx)

- **400 Bad Request**: Invalid request format or parameters
- **401 Unauthorized**: Authentication required
- **403 Forbidden**: Access denied (authenticated but not authorized)
- **404 Not Found**: Resource does not exist
- **405 Method Not Allowed**: HTTP method not supported
- **409 Conflict**: Request conflicts with current state
- **422 Unprocessable Entity**: Validation errors

### Server Error Status Codes (5xx)

- **500 Internal Server Error**: Unhandled server error
- **502 Bad Gateway**: Upstream service error
- **503 Service Unavailable**: Service temporarily unavailable
- **504 Gateway Timeout**: Upstream service timeout

## Response Format

### Default Response Format

When no custom response is provided:
- **Content-Type**: `text/plain; charset=UTF-8`
- **Status**: The provided status code
- **Body**: The error message or default status text

### Custom Response Format

Full control over response headers, content type, and body:

```ts
const jsonErrorResponse = new Response(
  JSON.stringify({ 
    error: true,
    message: 'Validation failed',
    details: validationErrors 
  }),
  {
    status: 422,
    headers: { 'Content-Type': 'application/json' }
  }
)

throw new HTTPException(422, { res: jsonErrorResponse })
```

## Error Context Information

### Request Context in Errors

Access request information in error handlers:

```ts
app.onError((err, c) => {
  const errorInfo = {
    error: err.message,
    path: c.req.path,
    method: c.req.method,
    timestamp: new Date().toISOString()
  }
  
  if (err instanceof HTTPException) {
    return c.json(errorInfo, err.status)
  }
  
  return c.json({ ...errorInfo, error: 'Internal Server Error' }, 500)
})
```

## Best Practices

### Error Message Guidelines

- **Be specific but not revealing**: Include enough information to help the client without exposing internal details
- **Use consistent format**: Maintain consistent error message structure across your API
- **Localization consideration**: Structure messages for potential internationalization

### Security Considerations

- **Avoid information disclosure**: Don't reveal sensitive system information in error messages
- **Validate status codes**: Ensure status codes match the actual error condition
- **Log internal errors**: Use logging for detailed error tracking while keeping client responses clean

### Performance Considerations

- **Avoid expensive operations in error paths**: Keep error handling lightweight
- **Cache error responses**: For common errors, consider caching response objects
- **Minimize error object creation**: Reuse HTTPException instances when appropriate

## See Also

- [Hono Application Specification](/docs/api/hono) - Error handling configuration
- [Context Specification](/docs/api/context) - Error context and handling
- [Exception Examples](/docs/api/exception-examples) - Practical usage patterns
- [MDN Error.cause](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error/cause) - JavaScript Error cause property
