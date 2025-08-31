# Exception Handling Examples

Practical examples and patterns for error handling in Hono using HTTPException.

## Basic HTTPException Usage

### Simple Authentication Error

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
declare const authorized: boolean
// ---cut---
import { HTTPException } from 'hono/http-exception'

app.post('/auth', async (c, next) => {
  // authentication
  if (authorized === false) {
    throw new HTTPException(401, { message: 'Custom error message' })
  }
  await next()
})
```

### Validation Error

```ts twoslash
import { Hono } from 'hono'
import { HTTPException } from 'hono/http-exception'
const app = new Hono()
// ---cut---
app.post('/users', async (c) => {
  const body = await c.req.json()
  
  if (!body.email) {
    throw new HTTPException(400, { message: 'Email is required' })
  }
  
  if (!body.email.includes('@')) {
    throw new HTTPException(400, { message: 'Invalid email format' })
  }
  
  return c.json({ success: true })
})
```

## Custom Response Examples

### Custom Response with Headers

```ts twoslash
import { HTTPException } from 'hono/http-exception'

const errorResponse = new Response('Unauthorized', {
  status: 401,
  headers: {
    'WWW-Authenticate': 'Bearer error="invalid_token"',
    'Content-Type': 'text/plain'
  },
})

throw new HTTPException(401, { res: errorResponse })
```

### JSON Error Response

```ts twoslash
import { HTTPException } from 'hono/http-exception'

const jsonErrorResponse = new Response(
  JSON.stringify({
    error: 'Forbidden',
    code: 'INSUFFICIENT_PERMISSIONS',
    timestamp: new Date().toISOString()
  }),
  {
    status: 403,
    headers: { 'Content-Type': 'application/json' }
  }
)

throw new HTTPException(403, { res: jsonErrorResponse })
```

## Error Handling Examples

### Global Error Handler

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
import { HTTPException } from 'hono/http-exception'

app.onError((err, c) => {
  if (err instanceof HTTPException) {
    // Get the custom response
    return err.getResponse()
  }
  
  // Handle other errors
  console.error('Unexpected error:', err)
  return c.json({ error: 'Internal Server Error' }, 500)
})
```

### Specific Error Type Handling

```ts twoslash
import { Hono } from 'hono'
import { HTTPException } from 'hono/http-exception'
const app = new Hono()
// ---cut---
app.onError((err, c) => {
  if (err instanceof HTTPException) {
    // Handle HTTP exceptions
    const status = err.status
    
    if (status === 401) {
      return c.json({ error: 'Authentication required' }, 401)
    } else if (status === 403) {
      return c.json({ error: 'Access denied' }, 403)
    } else if (status >= 400 && status < 500) {
      return c.json({ error: 'Client error' }, status)
    }
    
    return err.getResponse()
  }
  
  // Handle non-HTTP errors
  return c.json({ error: 'Server error' }, 500)
})
```

## Error Cause Examples

### Wrapping Lower-Level Errors

```ts twoslash
import { Hono, Context } from 'hono'
import { HTTPException } from 'hono/http-exception'
const app = new Hono()
declare const message: string
declare const authorize: (c: Context) => void
// ---cut---
app.post('/auth', async (c, next) => {
  try {
    authorize(c)
  } catch (e) {
    throw new HTTPException(401, { message, cause: e })
  }
  await next()
})
```

### Database Error Wrapping

```ts twoslash
import { Hono } from 'hono'
import { HTTPException } from 'hono/http-exception'
const app = new Hono()
declare const db: { findUser: (id: string) => Promise<any> }
// ---cut---
app.get('/users/:id', async (c) => {
  const id = c.req.param('id')
  
  try {
    const user = await db.findUser(id)
    return c.json(user)
  } catch (dbError) {
    throw new HTTPException(500, { 
      message: 'Failed to retrieve user',
      cause: dbError 
    })
  }
})
```

## Middleware Error Patterns

### Authentication Middleware

```ts twoslash
import { Hono } from 'hono'
import { HTTPException } from 'hono/http-exception'
const app = new Hono()
declare const verifyToken: (token: string) => Promise<any>
// ---cut---
const authMiddleware = async (c: any, next: any) => {
  const token = c.req.header('Authorization')?.replace('Bearer ', '')
  
  if (!token) {
    throw new HTTPException(401, { message: 'Authorization token required' })
  }
  
  try {
    const user = await verifyToken(token)
    c.set('user', user)
    await next()
  } catch (error) {
    throw new HTTPException(401, { message: 'Invalid token', cause: error })
  }
}

app.use('/protected/*', authMiddleware)
```

### Rate Limiting Middleware

```ts twoslash
import { Hono } from 'hono'
import { HTTPException } from 'hono/http-exception'
const app = new Hono()
declare const isRateLimited: (ip: string) => boolean
declare const getClientIP: (c: any) => string
// ---cut---
const rateLimitMiddleware = async (c: any, next: any) => {
  const clientIP = getClientIP(c)
  
  if (isRateLimited(clientIP)) {
    const retryAfter = 60 // seconds
    const response = new Response('Too Many Requests', {
      status: 429,
      headers: {
        'Retry-After': retryAfter.toString(),
        'X-RateLimit-Limit': '100',
        'X-RateLimit-Remaining': '0'
      }
    })
    
    throw new HTTPException(429, { res: response })
  }
  
  await next()
}

app.use('*', rateLimitMiddleware)
```

## Validation Error Patterns

### Form Validation

```ts twoslash
import { Hono } from 'hono'
import { HTTPException } from 'hono/http-exception'
const app = new Hono()
// ---cut---
app.post('/contact', async (c) => {
  const body = await c.req.parseBody()
  const errors: string[] = []
  
  if (!body.name) errors.push('Name is required')
  if (!body.email) errors.push('Email is required')
  if (!body.message) errors.push('Message is required')
  
  if (errors.length > 0) {
    const errorResponse = new Response(
      JSON.stringify({ errors }),
      {
        status: 400,
        headers: { 'Content-Type': 'application/json' }
      }
    )
    throw new HTTPException(400, { res: errorResponse })
  }
  
  return c.json({ success: true })
})
```

### Schema Validation

```ts twoslash
import { Hono } from 'hono'
import { HTTPException } from 'hono/http-exception'
const app = new Hono()
declare const validateSchema: (data: any) => { valid: boolean; errors?: string[] }
// ---cut---
app.post('/api/data', async (c) => {
  const body = await c.req.json()
  const validation = validateSchema(body)
  
  if (!validation.valid) {
    throw new HTTPException(400, { 
      message: 'Validation failed',
      res: new Response(JSON.stringify({
        error: 'Validation failed',
        details: validation.errors
      }), {
        status: 400,
        headers: { 'Content-Type': 'application/json' }
      })
    })
  }
  
  return c.json({ success: true })
})
```

## Resource Not Found Patterns

### Custom 404 Responses

```ts twoslash
import { Hono } from 'hono'
import { HTTPException } from 'hono/http-exception'
const app = new Hono()
declare const db: { findPost: (id: string) => Promise<any | null> }
// ---cut---
app.get('/posts/:id', async (c) => {
  const id = c.req.param('id')
  const post = await db.findPost(id)
  
  if (!post) {
    throw new HTTPException(404, { message: 'Post not found' })
  }
  
  return c.json(post)
})
```

### Resource Access Control

```ts
import { Hono } from 'hono'
import { HTTPException } from 'hono/http-exception'
const app = new Hono()
declare const canAccess: (userId: string, resourceId: string) => boolean

app.get('/documents/:id', async (c) => {
  const documentId = c.req.param('id')
  const user = c.get('user') as { id: string } | undefined
  
  if (!user || !canAccess(user.id, documentId)) {
    throw new HTTPException(403, { 
      message: 'Access denied to this document' 
    })
  }
  
  return c.json({ document: 'content' })
})
```

## Error Recovery Patterns

### Graceful Fallbacks

```ts twoslash
import { Hono } from 'hono'
import { HTTPException } from 'hono/http-exception'
const app = new Hono()
declare const primaryService: { getData: () => Promise<any> }
declare const fallbackService: { getData: () => Promise<any> }
// ---cut---
app.get('/data', async (c) => {
  try {
    const data = await primaryService.getData()
    return c.json(data)
  } catch (primaryError) {
    try {
      const fallbackData = await fallbackService.getData()
      return c.json(fallbackData)
    } catch (fallbackError) {
      throw new HTTPException(503, { 
        message: 'Service temporarily unavailable',
        cause: { primary: primaryError, fallback: fallbackError }
      })
    }
  }
})
```

### Retry Logic with Error

```ts
import { Hono } from 'hono'
import { HTTPException } from 'hono/http-exception'
const app = new Hono()
declare const unreliableService: { call: () => Promise<any> }

app.get('/external-data', async (c) => {
  const maxRetries = 3
  let lastError: Error | undefined
  
  for (let i = 0; i < maxRetries; i++) {
    try {
      const data = await unreliableService.call()
      return c.json(data)
    } catch (error) {
      lastError = error as Error
      if (i === maxRetries - 1) break
      
      // Wait before retry
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)))
    }
  }
  
  throw new HTTPException(502, { 
    message: 'External service unavailable after retries',
    cause: lastError
  })
})
```

## See Also

- [Exception Specification](/docs/api/exception) - Complete formal specification
- [Context Examples](/docs/api/context-examples) - Error response patterns
- [Hono Examples](/docs/api/hono-examples) - Global error handler setup