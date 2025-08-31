# Exception Examples

Practical examples demonstrating HTTP exception handling patterns in Hono applications.

## Basic HTTPException Usage

### Simple Status Code Exceptions

```ts twoslash
import { Hono } from 'hono'
import { HTTPException } from 'hono/http-exception'
const app = new Hono()
// ---cut---
// Basic exception with just status code
app.get('/protected', (c) => {
  const isAuthenticated = false // Your auth logic here
  
  if (!isAuthenticated) {
    throw new HTTPException(401) // Uses default message "Unauthorized"
  }
  
  return c.text('Protected content')
})

// Exception with custom message
app.get('/user/:id', async (c) => {
  const id = c.req.param('id')
  
  if (!id || isNaN(parseInt(id))) {
    throw new HTTPException(400, { message: 'Invalid user ID format' })
  }
  
  const user = await findUser(id)
  if (!user) {
    throw new HTTPException(404, { message: 'User not found' })
  }
  
  return c.json(user)
})

declare const findUser: (id: string) => Promise<any>
```

### Custom Response Exceptions

```ts twoslash
import { HTTPException } from 'hono/http-exception'
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// JSON error response
const jsonErrorResponse = new Response(
  JSON.stringify({
    error: 'UNAUTHORIZED',
    message: 'Invalid or expired token',
    code: 'AUTH001'
  }),
  {
    status: 401,
    headers: {
      'Content-Type': 'application/json',
      'WWW-Authenticate': 'Bearer realm="api"'
    }
  }
)

app.get('/api/profile', (c) => {
  const token = c.req.header('Authorization')
  
  if (!token) {
    throw new HTTPException(401, { res: jsonErrorResponse })
  }
  
  return c.json({ profile: 'user data' })
})

// Custom HTML error page
const htmlErrorResponse = new Response(
  `<!DOCTYPE html>
   <html>
   <body>
     <h1>Access Denied</h1>
     <p>You don't have permission to access this resource.</p>
   </body>
   </html>`,
  {
    status: 403,
    headers: { 'Content-Type': 'text/html' }
  }
)

app.get('/admin/*', (c) => {
  const isAdmin = false // Your admin check logic
  
  if (!isAdmin) {
    throw new HTTPException(403, { res: htmlErrorResponse })
  }
  
  return c.text('Admin content')
})
```

## Authentication and Authorization

### JWT Authentication Middleware

```ts
import { Hono } from 'hono'
import { HTTPException } from 'hono/http-exception'

type Variables = {
  user: any
}

const app = new Hono<{ Variables: Variables }>()

declare const verifyJWT: (token: string) => Promise<any>
// ---cut---
// JWT authentication middleware
app.use('/protected/*', async (c, next) => {
  const authHeader = c.req.header('Authorization')
  
  if (!authHeader) {
    throw new HTTPException(401, { 
      message: 'Authorization header is required' 
    })
  }
  
  const token = authHeader.replace('Bearer ', '')
  
  if (!token) {
    throw new HTTPException(401, { 
      message: 'Bearer token is required' 
    })
  }
  
  try {
    const payload = await verifyJWT(token)
    c.set('user', payload)
  } catch (error) {
    throw new HTTPException(401, { 
      message: 'Invalid or expired token',
      cause: error
    })
  }
  
  await next()
})

app.get('/protected/data', (c) => {
  const user = c.get('user')
  return c.json({ message: `Hello, ${user.name}` })
})
```

### Role-Based Authorization

```ts twoslash
import { Hono } from 'hono'
import { HTTPException } from 'hono/http-exception'

const app = new Hono()

interface User {
  id: string
  role: string
  permissions: string[]
}

declare const getCurrentUser: () => User | null
// ---cut---
// Role-based authorization
const requireRole = (requiredRole: string) => {
  return async (c: any, next: any) => {
    const user = getCurrentUser()
    
    if (!user) {
      throw new HTTPException(401, { message: 'Authentication required' })
    }
    
    if (user.role !== requiredRole) {
      throw new HTTPException(403, { 
        message: `Access denied. Required role: ${requiredRole}` 
      })
    }
    
    await next()
  }
}

// Permission-based authorization
const requirePermission = (permission: string) => {
  return async (c: any, next: any) => {
    const user = getCurrentUser()
    
    if (!user) {
      throw new HTTPException(401, { message: 'Authentication required' })
    }
    
    if (!user.permissions.includes(permission)) {
      throw new HTTPException(403, { 
        message: `Access denied. Required permission: ${permission}` 
      })
    }
    
    await next()
  }
}

// Using authorization middleware
app.get('/admin/users', requireRole('admin'), (c) => {
  return c.json({ users: ['admin users'] })
})

app.delete('/posts/:id', requirePermission('posts:delete'), (c) => {
  const id = c.req.param('id')
  return c.json({ message: `Deleted post ${id}` })
})
```

## Input Validation Errors

### Form Validation

```ts twoslash
import { Hono } from 'hono'
import { HTTPException } from 'hono/http-exception'

const app = new Hono()
// ---cut---
app.post('/register', async (c) => {
  const body = await c.req.json()
  const errors: string[] = []
  
  // Validation logic
  if (!body.email || !body.email.includes('@')) {
    errors.push('Valid email is required')
  }
  
  if (!body.password || body.password.length < 8) {
    errors.push('Password must be at least 8 characters')
  }
  
  if (!body.name || body.name.trim().length < 2) {
    errors.push('Name must be at least 2 characters')
  }
  
  if (errors.length > 0) {
    const errorResponse = new Response(
      JSON.stringify({
        error: 'Validation failed',
        errors: errors
      }),
      {
        status: 422,
        headers: { 'Content-Type': 'application/json' }
      }
    )
    
    throw new HTTPException(422, { res: errorResponse })
  }
  
  return c.json({ message: 'User registered successfully' })
})
```

### Query Parameter Validation

```ts twoslash
import { Hono } from 'hono'
import { HTTPException } from 'hono/http-exception'

const app = new Hono()
// ---cut---
app.get('/search', (c) => {
  const query = c.req.query('q')
  const limit = c.req.query('limit')
  const offset = c.req.query('offset')
  
  if (!query || query.trim().length === 0) {
    throw new HTTPException(400, { 
      message: 'Search query "q" parameter is required' 
    })
  }
  
  if (query.length < 3) {
    throw new HTTPException(400, { 
      message: 'Search query must be at least 3 characters long' 
    })
  }
  
  const parsedLimit = limit ? parseInt(limit) : 10
  const parsedOffset = offset ? parseInt(offset) : 0
  
  if (isNaN(parsedLimit) || parsedLimit < 1 || parsedLimit > 100) {
    throw new HTTPException(400, { 
      message: 'Limit must be a number between 1 and 100' 
    })
  }
  
  if (isNaN(parsedOffset) || parsedOffset < 0) {
    throw new HTTPException(400, { 
      message: 'Offset must be a non-negative number' 
    })
  }
  
  return c.json({ 
    query,
    limit: parsedLimit,
    offset: parsedOffset,
    results: []
  })
})
```

## Resource Management Errors

### Database Errors

```ts twoslash
import { Hono } from 'hono'
import { HTTPException } from 'hono/http-exception'

const app = new Hono()

class DatabaseError extends Error {
  constructor(message: string) {
    super(message)
    this.name = 'DatabaseError'
  }
}

declare const db: {
  findUser: (id: string) => Promise<any>
  createUser: (data: any) => Promise<any>
  updateUser: (id: string, data: any) => Promise<any>
}
// ---cut---
app.get('/users/:id', async (c) => {
  const id = c.req.param('id')
  
  try {
    const user = await db.findUser(id)
    
    if (!user) {
      throw new HTTPException(404, { message: 'User not found' })
    }
    
    return c.json(user)
  } catch (error) {
    if (error instanceof HTTPException) {
      throw error // Re-throw HTTPExceptions
    }
    
    // Handle database errors
    console.error('Database error:', error)
    throw new HTTPException(500, { 
      message: 'Internal server error',
      cause: error
    })
  }
})

app.post('/users', async (c) => {
  try {
    const userData = await c.req.json()
    const newUser = await db.createUser(userData)
    
    return c.json(newUser, 201)
  } catch (error) {
    if (error instanceof DatabaseError) {
      if (error.message.includes('duplicate key')) {
        throw new HTTPException(409, { 
          message: 'User with this email already exists' 
        })
      }
    }
    
    throw new HTTPException(500, { 
      message: 'Failed to create user',
      cause: error
    })
  }
})
```

### File Upload Errors

```ts twoslash
import { Hono } from 'hono'
import { HTTPException } from 'hono/http-exception'

const app = new Hono()
// ---cut---
app.post('/upload', async (c) => {
  try {
    const body = await c.req.parseBody()
    const file = body.file as File
    
    if (!file) {
      throw new HTTPException(400, { message: 'No file uploaded' })
    }
    
    // Check file size (5MB limit)
    const maxSize = 5 * 1024 * 1024
    if (file.size > maxSize) {
      throw new HTTPException(413, { 
        message: `File too large. Maximum size is ${maxSize / 1024 / 1024}MB` 
      })
    }
    
    // Check file type
    const allowedTypes = ['image/jpeg', 'image/png', 'image/gif']
    if (!allowedTypes.includes(file.type)) {
      throw new HTTPException(415, { 
        message: `Unsupported file type. Allowed types: ${allowedTypes.join(', ')}` 
      })
    }
    
    // Process file upload
    return c.json({ message: 'File uploaded successfully', filename: file.name })
  } catch (error) {
    if (error instanceof HTTPException) {
      throw error
    }
    
    throw new HTTPException(500, { 
      message: 'File upload failed',
      cause: error
    })
  }
})
```

## Rate Limiting and Throttling

### Rate Limit Exceptions

```ts twoslash
import { Hono } from 'hono'
import { HTTPException } from 'hono/http-exception'

const app = new Hono()

// Simple in-memory rate limiter (use Redis in production)
const rateLimiter = new Map<string, { count: number; resetTime: number }>()

const RATE_LIMIT = 100 // requests per hour
const WINDOW_MS = 60 * 60 * 1000 // 1 hour in milliseconds
// ---cut---
app.use('/api/*', async (c, next) => {
  const clientIP = c.req.header('x-forwarded-for') || 'unknown'
  const now = Date.now()
  const windowStart = now - WINDOW_MS
  
  let clientData = rateLimiter.get(clientIP)
  
  if (!clientData || clientData.resetTime < windowStart) {
    clientData = { count: 0, resetTime: now + WINDOW_MS }
  }
  
  clientData.count++
  rateLimiter.set(clientIP, clientData)
  
  if (clientData.count > RATE_LIMIT) {
    const resetTime = new Date(clientData.resetTime).toISOString()
    
    const rateLimitResponse = new Response(
      JSON.stringify({
        error: 'Rate limit exceeded',
        message: `Too many requests. Limit: ${RATE_LIMIT} per hour`,
        resetTime: resetTime
      }),
      {
        status: 429,
        headers: {
          'Content-Type': 'application/json',
          'X-RateLimit-Limit': RATE_LIMIT.toString(),
          'X-RateLimit-Remaining': '0',
          'X-RateLimit-Reset': clientData.resetTime.toString(),
          'Retry-After': Math.ceil((clientData.resetTime - now) / 1000).toString()
        }
      }
    )
    
    throw new HTTPException(429, { res: rateLimitResponse })
  }
  
  await next()
})
```

## Error Handling Patterns

### Global Error Handler

```ts twoslash
import { Hono } from 'hono'
import { HTTPException } from 'hono/http-exception'

const app = new Hono()
// ---cut---
app.onError((err, c) => {
  // Log all errors for monitoring
  console.error(`Error: ${err.message}`, {
    path: c.req.path,
    method: c.req.method,
    userAgent: c.req.header('user-agent'),
    timestamp: new Date().toISOString(),
    stack: err.stack
  })
  
  // Handle HTTPException
  if (err instanceof HTTPException) {
    return err.getResponse()
  }
  
  // Handle specific error types
  if (err.name === 'ValidationError') {
    return c.json({ 
      error: 'Validation failed',
      message: err.message 
    }, 400)
  }
  
  if (err.name === 'DatabaseConnectionError') {
    return c.json({ 
      error: 'Service temporarily unavailable',
      message: 'Please try again later'
    }, 503)
  }
  
  // Default error response
  const isDevelopment = process.env.NODE_ENV === 'development'
  
  return c.json({
    error: 'Internal Server Error',
    message: isDevelopment ? err.message : 'Something went wrong',
    ...(isDevelopment && { stack: err.stack })
  }, 500)
})
```

### Middleware Error Handling

```ts twoslash
import { Hono } from 'hono'
import { HTTPException } from 'hono/http-exception'

const app = new Hono()
// ---cut---
// Error handling middleware
const errorHandler = async (c: any, next: any) => {
  try {
    await next()
  } catch (error) {
    // Add context information to errors
    if (error instanceof HTTPException) {
      // Add request context to HTTPException
      const enhancedResponse = new Response(
        JSON.stringify({
          error: error.message,
          path: c.req.path,
          method: c.req.method,
          timestamp: new Date().toISOString()
        }),
        {
          status: error.status,
          headers: { 'Content-Type': 'application/json' }
        }
      )
      
      throw new HTTPException(error.status, { res: enhancedResponse })
    }
    
    // Re-throw other errors
    throw error
  }
}

app.use('*', errorHandler)
```

## Testing Exception Handling

### Unit Testing HTTPException

```ts twoslash
import { Hono } from 'hono'
import { HTTPException } from 'hono/http-exception'

declare const test: (name: string, fn: () => void) => void
declare const expect: (value: any) => any

const app = new Hono()

app.get('/test-auth', (c) => {
  const token = c.req.header('Authorization')
  if (!token) {
    throw new HTTPException(401, { message: 'Token required' })
  }
  return c.text('Authenticated')
})
// ---cut---
test('throws 401 when no authorization header', async () => {
  const res = await app.request('/test-auth')
  expect(res.status).toBe(401)
  
  const text = await res.text()
  expect(text).toBe('Token required')
})

test('returns 200 when authorized', async () => {
  const res = await app.request('/test-auth', {
    headers: { Authorization: 'Bearer token123' }
  })
  expect(res.status).toBe(200)
  expect(await res.text()).toBe('Authenticated')
})
```

### Integration Testing with Error Scenarios

```ts twoslash
import { Hono } from 'hono'
import { HTTPException } from 'hono/http-exception'

declare const test: (name: string, fn: () => void) => void
declare const expect: (value: any) => any

const app = new Hono()

app.post('/users', async (c) => {
  const body = await c.req.json()
  
  if (!body.email) {
    throw new HTTPException(400, { message: 'Email is required' })
  }
  
  if (body.email === 'exists@example.com') {
    throw new HTTPException(409, { message: 'Email already exists' })
  }
  
  return c.json({ id: 1, email: body.email }, 201)
})
// ---cut---
test('user creation validation errors', async () => {
  // Test missing email
  const res1 = await app.request('/users', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({})
  })
  
  expect(res1.status).toBe(400)
  expect(await res1.text()).toBe('Email is required')
  
  // Test duplicate email
  const res2 = await app.request('/users', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email: 'exists@example.com' })
  })
  
  expect(res2.status).toBe(409)
  expect(await res2.text()).toBe('Email already exists')
  
  // Test successful creation
  const res3 = await app.request('/users', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email: 'new@example.com' })
  })
  
  expect(res3.status).toBe(201)
  const user = await res3.json()
  expect(user.email).toBe('new@example.com')
})
```

## Best Practices Examples

### Consistent Error Response Format

```ts twoslash
import { HTTPException } from 'hono/http-exception'

// ---cut---
// Utility function for consistent error responses
const createErrorResponse = (status: number, code: string, message: string, details?: any) => {
  const errorBody = {
    error: true,
    code,
    message,
    timestamp: new Date().toISOString(),
    ...(details && { details })
  }
  
  return new Response(JSON.stringify(errorBody), {
    status,
    headers: { 'Content-Type': 'application/json' }
  })
}

// Usage examples
throw new HTTPException(400, { 
  res: createErrorResponse(400, 'INVALID_INPUT', 'Invalid user input', {
    field: 'email',
    reason: 'Email format is invalid'
  })
})

throw new HTTPException(404, {
  res: createErrorResponse(404, 'RESOURCE_NOT_FOUND', 'User not found')
})

throw new HTTPException(429, {
  res: createErrorResponse(429, 'RATE_LIMIT_EXCEEDED', 'Too many requests')
})
```

### Error Recovery Patterns

```ts twoslash
import { Hono } from 'hono'
import { HTTPException } from 'hono/http-exception'

const app = new Hono()

declare const primaryDatabase: any
declare const fallbackDatabase: any
// ---cut---
// Graceful degradation example
app.get('/users/:id', async (c) => {
  const id = c.req.param('id')
  
  try {
    // Try primary data source
    const user = await primaryDatabase.findUser(id)
    return c.json(user)
  } catch (primaryError) {
    try {
      // Fallback to secondary data source
      console.warn('Primary database failed, using fallback')
      const user = await fallbackDatabase.findUser(id)
      
      return c.json(user, 200, {
        'X-Data-Source': 'fallback'
      })
    } catch (fallbackError) {
      // Both failed, return error
      throw new HTTPException(503, {
        message: 'Service temporarily unavailable',
        cause: { primary: primaryError, fallback: fallbackError }
      })
    }
  }
})
```

## See Also

- [Exception Specification](/docs/api/exception) - Complete HTTPException API reference
- [Hono Examples](/docs/api/hono-examples) - Application error handling setup
- [Context Examples](/docs/api/context-examples) - Context-based error handling
- [Middleware Guide](/docs/guides/middleware) - Error handling middleware patterns