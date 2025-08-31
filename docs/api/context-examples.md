# Context Usage Examples

This page provides practical examples for working with the Context object in Hono applications.

## Response Generation Examples

### Text Responses

```ts
import { Hono } from 'hono'

const app = new Hono()

app.get('/hello', (c) => {
  return c.text('Hello World!')
})

app.get('/hello/:name', (c) => {
  const name = c.req.param('name')
  return c.text(`Hello ${name}!`, 200, {
    'X-Custom-Header': 'greeting'
  })
})
```

### JSON Responses

```ts
app.get('/users', (c) => {
  return c.json({
    users: [
      { id: 1, name: 'Alice' },
      { id: 2, name: 'Bob' }
    ]
  })
})

app.post('/users', async (c) => {
  const body = await c.req.json()
  
  return c.json({
    message: 'User created',
    user: { id: 123, ...body }
  }, 201)
})
```

### HTML Responses

```ts
app.get('/welcome', (c) => {
  const html = `
    <!DOCTYPE html>
    <html>
    <head><title>Welcome</title></head>
    <body><h1>Welcome to Hono!</h1></body>
    </html>
  `
  return c.html(html)
})
```

### Binary Responses

```ts
app.get('/download', async (c) => {
  const fileData = new Uint8Array([1, 2, 3, 4])
  
  return c.body(fileData, 200, {
    'Content-Type': 'application/octet-stream',
    'Content-Disposition': 'attachment; filename="data.bin"'
  })
})
```

### Redirect Responses

```ts
app.get('/old-path', (c) => {
  return c.redirect('/new-path')
})

app.get('/external', (c) => {
  return c.redirect('https://example.com', 302)
})
```

## Environment Access Examples

### Cloudflare Workers Environment

```ts
type Env = {
  API_KEY: string
  MY_KV: KVNamespace
  DB: D1Database
}

const app = new Hono<{ Bindings: Env }>()

app.get('/data/:key', async (c) => {
  // Access KV store
  const key = c.req.param('key')
  const value = await c.env.MY_KV.get(key)
  
  if (!value) {
    return c.text('Not found', 404)
  }
  
  return c.text(value)
})

app.get('/secret', (c) => {
  // Access environment secret
  const apiKey = c.env.API_KEY
  return c.json({ 
    hasApiKey: !!apiKey,
    keyLength: apiKey.length 
  })
})
```

### Database Access

```ts
app.get('/users/:id', async (c) => {
  const id = c.req.param('id')
  
  // Use database binding
  const result = await c.env.DB.prepare('SELECT * FROM users WHERE id = ?')
    .bind(id)
    .first()
  
  if (!result) {
    return c.text('User not found', 404)
  }
  
  return c.json({ user: result })
})
```

## Variable Storage Examples

### Request Scoped Variables

```ts
type Variables = {
  user: { id: string; name: string }
  startTime: number
}

const app = new Hono<{ Variables: Variables }>()

// Middleware to set user
app.use('/protected/*', async (c, next) => {
  // Simulate user authentication
  const user = { id: '123', name: 'Alice' }
  c.set('user', user)
  
  // Set request start time
  c.set('startTime', Date.now())
  
  await next()
})

app.get('/protected/profile', (c) => {
  const user = c.get('user')       // Type-safe access
  const startTime = c.get('startTime')
  
  return c.json({
    profile: user,
    processingTime: Date.now() - startTime
  })
})
```

### Middleware Communication

```ts
// Auth middleware sets user info
app.use('/api/*', async (c, next) => {
  const token = c.req.header('Authorization')
  
  if (token) {
    const user = await validateToken(token)
    c.set('user', user)
  }
  
  await next()
})

// Rate limiting middleware uses user info
app.use('/api/*', async (c, next) => {
  const user = c.get('user')
  
  if (user) {
    // Premium users get higher rate limits
    c.set('rateLimit', user.isPremium ? 1000 : 100)
  } else {
    c.set('rateLimit', 10)
  }
  
  await next()
})

app.get('/api/data', (c) => {
  const user = c.get('user')
  const rateLimit = c.get('rateLimit')
  
  return c.json({
    user: user?.name || 'anonymous',
    rateLimit
  })
})

async function validateToken(token: string) {
  // Mock validation
  return { id: '123', name: 'Alice', isPremium: true }
}
```

## Header Management Examples

### Setting Response Headers

```ts
app.get('/api/data', (c) => {
  // Set individual header
  c.header('X-API-Version', '1.0')
  
  // Set multiple headers
  c.header('Cache-Control', 'max-age=3600')
  c.header('X-Content-Type-Options', 'nosniff')
  
  return c.json({ data: 'example' })
})
```

### Conditional Headers

```ts
app.get('/download/:file', async (c) => {
  const filename = c.req.param('file')
  const userAgent = c.req.header('User-Agent')
  
  // Set headers based on client
  if (userAgent?.includes('Mobile')) {
    c.header('X-Mobile-Optimized', 'true')
  }
  
  c.header('Content-Disposition', `attachment; filename="${filename}"`)
  
  return c.text('File content here')
})
```

### CORS Headers

```ts
app.use('/api/*', async (c, next) => {
  // Set CORS headers
  c.header('Access-Control-Allow-Origin', '*')
  c.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE')
  c.header('Access-Control-Allow-Headers', 'Content-Type, Authorization')
  
  if (c.req.method === 'OPTIONS') {
    return c.text('', 204)
  }
  
  await next()
})
```

## Status Code Examples

### REST API Status Codes

```ts
// GET - Resource found
app.get('/users/:id', async (c) => {
  const user = await findUser(c.req.param('id'))
  
  if (!user) {
    return c.text('User not found', 404)
  }
  
  return c.json(user, 200)
})

// POST - Resource created
app.post('/users', async (c) => {
  const body = await c.req.json()
  const user = await createUser(body)
  
  return c.json(user, 201)
})

// PUT - Resource updated
app.put('/users/:id', async (c) => {
  const id = c.req.param('id')
  const body = await c.req.json()
  
  const updated = await updateUser(id, body)
  if (!updated) {
    return c.text('User not found', 404)
  }
  
  return c.json(updated, 200)
})

// DELETE - Resource deleted
app.delete('/users/:id', async (c) => {
  const id = c.req.param('id')
  const deleted = await deleteUser(id)
  
  if (!deleted) {
    return c.text('User not found', 404)
  }
  
  return c.text('', 204) // No content
})

// Mock functions
async function findUser(id: string) { return { id, name: 'User' } }
async function createUser(data: any) { return { id: '123', ...data } }
async function updateUser(id: string, data: any) { return { id, ...data } }
async function deleteUser(id: string) { return true }
```

## Content Negotiation Examples

### Multiple Response Formats

```ts
app.get('/users/:id', async (c) => {
  const id = c.req.param('id')
  const accept = c.req.header('Accept')
  const user = { id, name: 'Alice', email: 'alice@example.com' }
  
  if (accept?.includes('application/xml')) {
    const xml = `
      <user>
        <id>${user.id}</id>
        <name>${user.name}</name>
        <email>${user.email}</email>
      </user>
    `
    return c.text(xml.trim(), 200, { 'Content-Type': 'application/xml' })
  }
  
  if (accept?.includes('text/csv')) {
    const csv = `id,name,email\n${user.id},${user.name},${user.email}`
    return c.text(csv, 200, { 'Content-Type': 'text/csv' })
  }
  
  // Default to JSON
  return c.json(user)
})
```

## Rendering Examples

### Template Rendering

```ts
app.get('/user/:name', (c) => {
  const name = c.req.param('name')
  
  return c.html(`
    <html>
      <body>
        <h1>Hello ${name}!</h1>
        <p>Welcome to our site.</p>
      </body>
    </html>
  `)
})
```

### Dynamic Content Generation

```ts
app.get('/report/:type', async (c) => {
  const type = c.req.param('type')
  const format = c.req.query('format') || 'json'
  
  const data = await generateReport(type)
  
  if (format === 'html') {
    const html = `
      <html>
        <body>
          <h1>${type} Report</h1>
          <pre>${JSON.stringify(data, null, 2)}</pre>
        </body>
      </html>
    `
    return c.html(html)
  }
  
  return c.json(data)
})

async function generateReport(type: string) {
  return { type, timestamp: new Date().toISOString(), data: [] }
}
```

## Error Context Examples

### Error Information Access

```ts
import { HTTPException } from 'hono/http-exception'

app.use('*', async (c, next) => {
  try {
    await next()
  } catch (error) {
    console.error('Request failed:', {
      path: c.req.path,
      method: c.req.method,
      error: error.message
    })
    throw error
  }
})

app.get('/error-prone/:id', (c) => {
  const id = c.req.param('id')
  
  if (id === 'bad') {
    throw new HTTPException(400, { 
      message: 'Invalid ID provided',
      cause: new Error('ID validation failed')
    })
  }
  
  return c.json({ id })
})
```

### Error Recovery

```ts
app.use('/api/*', async (c, next) => {
  try {
    await next()
  } catch (error) {
    // Store error in context for later handling
    c.set('lastError', error)
    
    if (error instanceof HTTPException) {
      return error.getResponse()
    }
    
    return c.json({ 
      error: 'Internal server error',
      requestId: c.req.header('X-Request-ID') || 'unknown'
    }, 500)
  }
})
```

## Platform Integration Examples

### Cloudflare Workers Integration

```ts
type Env = {
  KV: KVNamespace
  SECRET: string
}

const app = new Hono<{ Bindings: Env }>()

app.get('/cache/:key', async (c) => {
  const key = c.req.param('key')
  const value = await c.env.KV.get(key)
  
  if (!value) {
    return c.text('Not found', 404)
  }
  
  return c.text(value)
})

app.put('/cache/:key', async (c) => {
  const key = c.req.param('key')
  const value = await c.req.text()
  
  await c.env.KV.put(key, value)
  
  return c.text('Stored')
})
```

### Deno Integration

```ts
// Access Deno-specific features
app.get('/system-info', (c) => {
  return c.json({
    platform: Deno?.build?.os || 'unknown',
    arch: Deno?.build?.arch || 'unknown',
    version: Deno?.version?.deno || 'unknown'
  })
})
```

## Stream and Response Examples

### Streaming Response

```ts
app.get('/stream', (c) => {
  const stream = new ReadableStream({
    start(controller) {
      // Send data chunks
      controller.enqueue('chunk 1\n')
      setTimeout(() => {
        controller.enqueue('chunk 2\n')
        controller.close()
      }, 1000)
    }
  })
  
  return c.stream(stream)
})
```

### File Download

```ts
app.get('/download/:filename', async (c) => {
  const filename = c.req.param('filename')
  
  // Mock file content
  const content = `This is the content of ${filename}`
  
  return c.text(content, 200, {
    'Content-Type': 'application/octet-stream',
    'Content-Disposition': `attachment; filename="${filename}"`
  })
})
```

## Cookie Management Examples

### Setting Cookies

```ts
import { getCookie, setCookie, deleteCookie } from 'hono/cookie'

app.post('/login', async (c) => {
  const { username, password } = await c.req.json()
  
  // Validate credentials
  if (await validateLogin(username, password)) {
    setCookie(c, 'session', 'abc123', {
      httpOnly: true,
      secure: true,
      sameSite: 'Strict',
      maxAge: 3600
    })
    
    return c.json({ message: 'Login successful' })
  }
  
  return c.json({ error: 'Invalid credentials' }, 401)
})

app.get('/profile', (c) => {
  const sessionId = getCookie(c, 'session')
  
  if (!sessionId) {
    return c.text('Not logged in', 401)
  }
  
  return c.json({ user: 'Alice' })
})

app.post('/logout', (c) => {
  deleteCookie(c, 'session')
  return c.text('Logged out')
})

async function validateLogin(username: string, password: string) {
  return username === 'admin' && password === 'secret'
}
```

## Middleware Context Examples

### Request Logging

```ts
app.use('*', async (c, next) => {
  const start = Date.now()
  
  await next()
  
  const ms = Date.now() - start
  console.log(`${c.req.method} ${c.req.path} - ${ms}ms`)
})
```

### Authentication Middleware

```ts
interface User {
  id: string
  name: string
  role: string
}

app.use('/admin/*', async (c, next) => {
  const token = c.req.header('Authorization')?.replace('Bearer ', '')
  
  if (!token) {
    return c.text('Missing token', 401)
  }
  
  const user = await validateToken(token)
  if (!user || user.role !== 'admin') {
    return c.text('Forbidden', 403)
  }
  
  c.set('user', user)
  await next()
})

app.get('/admin/users', (c) => {
  const user = c.get('user') as User
  
  return c.json({
    message: `Hello ${user.name}`,
    users: []
  })
})

async function validateToken(token: string): Promise<User | null> {
  // Mock validation
  return token === 'valid-token' 
    ? { id: '1', name: 'Admin', role: 'admin' }
    : null
}
```

### Request Enrichment

```ts
app.use('/api/*', async (c, next) => {
  // Add request metadata
  c.set('requestId', crypto.randomUUID())
  c.set('timestamp', new Date().toISOString())
  c.set('clientIP', c.req.header('CF-Connecting-IP') || 'unknown')
  
  await next()
})

app.get('/api/info', (c) => {
  return c.json({
    requestId: c.get('requestId'),
    timestamp: c.get('timestamp'),
    clientIP: c.get('clientIP')
  })
})
```

## Error Handling Examples

### Custom Error Pages

```ts
app.onError((err, c) => {
  const accept = c.req.header('Accept')
  
  if (err instanceof HTTPException) {
    if (accept?.includes('application/json')) {
      return c.json({
        error: err.message,
        status: err.status
      }, err.status)
    } else {
      return c.html(`
        <html>
          <body>
            <h1>Error ${err.status}</h1>
            <p>${err.message}</p>
          </body>
        </html>
      `, err.status)
    }
  }
  
  // Generic error
  return c.text('Internal Server Error', 500)
})
```

### Error Context Information

```ts
app.use('*', async (c, next) => {
  try {
    await next()
  } catch (error) {
    // Log error with context
    console.error('Error occurred:', {
      path: c.req.path,
      method: c.req.method,
      headers: c.req.header(),
      error: error.message,
      stack: error.stack
    })
    
    throw error
  }
})
```

## Performance and Caching Examples

### Response Caching

```ts
app.use('/api/static/*', async (c, next) => {
  // Set cache headers for static API responses
  c.header('Cache-Control', 'public, max-age=3600')
  c.header('ETag', `"${Date.now()}"`)
  
  const ifNoneMatch = c.req.header('If-None-Match')
  if (ifNoneMatch) {
    return c.text('', 304) // Not Modified
  }
  
  await next()
})
```

### Conditional Processing

```ts
app.get('/heavy-computation/:id', async (c) => {
  const id = c.req.param('id')
  
  // Check if client wants cached version
  const acceptStale = c.req.header('Accept-Stale') === 'true'
  
  if (acceptStale) {
    const cached = await getCachedResult(id)
    if (cached) {
      c.header('X-Cache', 'HIT')
      return c.json(cached)
    }
  }
  
  // Perform computation
  const result = await expensiveComputation(id)
  await cacheResult(id, result)
  
  c.header('X-Cache', 'MISS')
  return c.json(result)
})

async function getCachedResult(id: string) {
  // Mock cache lookup
  return null
}

async function expensiveComputation(id: string) {
  // Mock computation
  return { id, computed: true, timestamp: Date.now() }
}

async function cacheResult(id: string, result: any) {
  // Mock cache storage
}
```

## Request Context Debugging

### Development Debugging

```ts
app.use('*', async (c, next) => {
  if (process.env.NODE_ENV === 'development') {
    console.log('Request Debug Info:', {
      url: c.req.url,
      method: c.req.method,
      headers: Object.fromEntries(
        Object.entries(c.req.header()).slice(0, 5) // Limit output
      ),
      timestamp: new Date().toISOString()
    })
  }
  
  await next()
})
```

### Request Tracing

```ts
app.use('*', async (c, next) => {
  const traceId = c.req.header('X-Trace-ID') || crypto.randomUUID()
  
  c.set('traceId', traceId)
  c.header('X-Trace-ID', traceId)
  
  console.log(`[${traceId}] ${c.req.method} ${c.req.path} - START`)
  
  try {
    await next()
    console.log(`[${traceId}] ${c.req.method} ${c.req.path} - SUCCESS`)
  } catch (error) {
    console.log(`[${traceId}] ${c.req.method} ${c.req.path} - ERROR: ${error.message}`)
    throw error
  }
})
```

## Type Extension Examples

### Custom Context Variables

```ts
declare module 'hono' {
  interface ContextVariableMap {
    user: { id: string; name: string; role: string }
    requestId: string
    startTime: number
    traceId: string
  }
}

// Now TypeScript knows about these variables
app.use('*', async (c, next) => {
  c.set('requestId', crypto.randomUUID())
  c.set('startTime', Date.now())
  await next()
})

app.get('/info', (c) => {
  // These are now fully typed
  const requestId = c.get('requestId')     // string
  const startTime = c.get('startTime')     // number
  
  return c.json({ requestId, duration: Date.now() - startTime })
})
```

## See Also

- [Context API Specification](/docs/api/context) - Complete Context API reference
- [HonoRequest Examples](/docs/api/request-examples) - Request parsing patterns
- [Hono Examples](/docs/api/hono-examples) - Application setup patterns
- [Middleware Guide](/docs/guides/middleware) - Middleware development patterns