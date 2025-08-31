# Exception Specification

Hono provides structured error handling through the `HTTPException` class for standardized HTTP error responses.

## HTTPException Class

### Constructor

```ts
new HTTPException(status, options?)
```

**Parameters:**

- `status`: `number` - HTTP status code (400-599)
- `options` (optional): `HTTPExceptionOptions`
  - `message?: string` - Error message
  - `res?: Response` - Custom response object
  - `cause?: unknown` - Error cause for debugging

**Returns:** `HTTPException` instance

### Properties

#### `status`

The HTTP status code for the exception.

**Type:** `number`

**Range:** 400-599 (client and server error codes)

#### `message`

The error message associated with the exception.

**Type:** `string`

**Default:** Standard HTTP status text for the status code

#### `cause`

Optional cause information for the exception.

**Type:** `unknown`

**Usage:** Debugging and error tracking

### Methods

#### `getResponse()`

Generate the HTTP response for this exception.

**Returns:** `Response`

**Notes:**

- If custom response was provided in constructor, returns that response
- Otherwise, generates response with status code and message
- Response body contains the error message

## Error Throwing

### Basic Exception Throwing

Throw an HTTPException to trigger error handling:

```ts
throw new HTTPException(status, { message: 'Error description' })
```

**Common Status Codes:**

- `400` - Bad Request
- `401` - Unauthorized
- `403` - Forbidden
- `404` - Not Found
- `422` - Unprocessable Entity
- `500` - Internal Server Error

### Custom Response Throwing

Provide a complete custom response:

```ts
const errorResponse = new Response('Custom error body', {
  status: 401,
  headers: { 'WWW-Authenticate': 'Bearer' },
})

throw new HTTPException(401, { res: errorResponse })
```

### Error Cause Tracking

Include cause information for debugging:

```ts
try {
  await dangerousOperation()
} catch (originalError) {
  throw new HTTPException(500, {
    message: 'Operation failed',
    cause: originalError,
  })
}
```

## Error Handling

### Global Error Handler

Handle HTTPExceptions globally with `app.onError()`:

```ts
app.onError((err, c) => {
  if (err instanceof HTTPException) {
    return err.getResponse()
  }
  return c.text('Internal Server Error', 500)
})
```

**Handler Signature:**

```ts
type ErrorHandler = (
  error: Error,
  c: Context
) => Response | Promise<Response>
```

### Error Response Customization

Customize error responses based on content type:

```ts
app.onError((err, c) => {
  if (err instanceof HTTPException) {
    const accept = c.req.header('Accept')

    if (accept?.includes('application/json')) {
      return c.json(
        {
          error: err.message,
          status: err.status,
        },
        err.status
      )
    }

    return err.getResponse() // Default response
  }

  return c.text('Internal Server Error', 500)
})
```

### Middleware Error Handling

Handle errors at the middleware level:

```ts
app.use('*', async (c, next) => {
  try {
    await next()
  } catch (err) {
    if (err instanceof HTTPException) {
      return err.getResponse()
    }
    throw err // Re-throw for global handler
  }
})
```

## Authentication Errors

### Authorization Failures

```ts
app.use('/protected/*', async (c, next) => {
  const token = c.req.header('Authorization')

  if (!token) {
    throw new HTTPException(401, {
      message: 'Authorization header required',
    })
  }

  if (!isValidToken(token)) {
    throw new HTTPException(401, {
      message: 'Invalid token',
    })
  }

  await next()
})
```

### Permission Errors

```ts
app.get('/admin/*', (c) => {
  const user = c.get('user')

  if (!user.isAdmin) {
    throw new HTTPException(403, {
      message: 'Admin access required',
    })
  }

  // Admin logic here
})
```

## Validation Errors

### Input Validation

```ts
app.post('/users', async (c) => {
  const body = await c.req.json()

  if (!body.email) {
    throw new HTTPException(400, {
      message: 'Email is required',
    })
  }

  if (!isValidEmail(body.email)) {
    throw new HTTPException(422, {
      message: 'Invalid email format',
    })
  }

  // Process valid input
})
```

### Resource Not Found

```ts
app.get('/users/:id', async (c) => {
  const id = c.req.param('id')
  const user = await findUser(id)

  if (!user) {
    throw new HTTPException(404, {
      message: 'User not found',
    })
  }

  return c.json(user)
})
```

## Server Errors

### Service Unavailable

```ts
app.get('/health', async (c) => {
  const dbHealthy = await checkDatabase()

  if (!dbHealthy) {
    throw new HTTPException(503, {
      message: 'Database unavailable',
    })
  }

  return c.json({ status: 'healthy' })
})
```

### Rate Limiting

```ts
app.use('/api/*', async (c, next) => {
  const rateLimitExceeded = await checkRateLimit(c.req)

  if (rateLimitExceeded) {
    throw new HTTPException(429, {
      message: 'Rate limit exceeded',
    })
  }

  await next()
})
```

## Error Context

### Request Information in Errors

```ts
app.onError((err, c) => {
  console.error('Error occurred:', {
    path: c.req.path,
    method: c.req.method,
    error: err.message,
    stack: err.stack,
  })

  if (err instanceof HTTPException) {
    return err.getResponse()
  }

  return c.text('Internal Server Error', 500)
})
```

### Error Logging

```ts
app.use('*', async (c, next) => {
  try {
    await next()
  } catch (err) {
    // Log error with context
    console.error('Request failed:', {
      timestamp: new Date().toISOString(),
      method: c.req.method,
      path: c.req.path,
      error: err.message,
      cause: err instanceof HTTPException ? err.cause : null,
    })

    throw err // Re-throw for error handler
  }
})
```

## Type Safety

### Custom Error Types

Extend HTTPException for specific error types:

```ts
class ValidationError extends HTTPException {
  constructor(field: string, message: string) {
    super(422, {
      message: `Validation error: ${field} - ${message}`,
    })
  }
}

class AuthenticationError extends HTTPException {
  constructor(message: string = 'Authentication required') {
    super(401, { message })
  }
}
```

### Error Handler Typing

Type error handlers for better development experience:

```ts
const errorHandler: ErrorHandler = (err, c) => {
  if (err instanceof HTTPException) {
    return err.getResponse()
  }

  return c.text('Server Error', 500)
}

app.onError(errorHandler)
```

## Best Practices

### Error Response Format

Maintain consistent error response format:

```ts
app.onError((err, c) => {
  if (err instanceof HTTPException) {
    return c.json(
      {
        error: {
          status: err.status,
          message: err.message,
          timestamp: new Date().toISOString(),
        },
      },
      err.status
    )
  }

  return c.json(
    {
      error: {
        status: 500,
        message: 'Internal Server Error',
        timestamp: new Date().toISOString(),
      },
    },
    500
  )
})
```

### Error Status Guidelines

Use appropriate HTTP status codes:

- **400 Bad Request** - Malformed request syntax
- **401 Unauthorized** - Authentication required
- **403 Forbidden** - Authenticated but insufficient permissions
- **404 Not Found** - Resource does not exist
- **422 Unprocessable Entity** - Valid syntax but semantic errors
- **429 Too Many Requests** - Rate limit exceeded
- **500 Internal Server Error** - Unexpected server error
- **503 Service Unavailable** - Temporary service outage

### Security Considerations

- Don't expose internal error details in production
- Log detailed errors server-side for debugging
- Use generic error messages for client responses
- Include request IDs for error correlation

## See Also

- [Context Specification](/docs/api/context) - Error handling in context
- [Hono Specification](/docs/api/hono) - Global error handler configuration
- [Middleware Guide](/docs/guides/middleware) - Error handling middleware patterns
