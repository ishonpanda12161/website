# Context Examples

Practical examples demonstrating how to use the Context object for request/response handling, variables, and platform-specific features.

## Request and Response Handling

### Basic Request Information

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/debug', (c) => {
  return c.json({
    method: c.req.method,
    path: c.req.path,
    url: c.req.url,
    userAgent: c.req.header('user-agent'),
    timestamp: new Date().toISOString()
  })
})
```

### Parameter Access

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/users/:id', (c) => {
  const id = c.req.param('id')
  return c.text(`User ID: ${id}`)
})

app.get('/posts/:id/comments/:commentId', (c) => {
  const { id, commentId } = c.req.param()
  return c.json({ postId: id, commentId })
})

app.get('/search', (c) => {
  const query = c.req.query('q')
  const limit = c.req.query('limit') || '10'
  const offset = c.req.query('offset') || '0'
  
  return c.json({ 
    query, 
    limit: parseInt(limit),
    offset: parseInt(offset)
  })
})
```

### Request Body Parsing

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// JSON body
app.post('/api/users', async (c) => {
  const user = await c.req.json()
  return c.json({ message: 'User created', user })
})

// Form data
app.post('/contact', async (c) => {
  const body = await c.req.parseBody()
  const name = body['name'] as string
  const email = body['email'] as string
  const message = body['message'] as string
  
  return c.json({ 
    message: 'Contact form received',
    data: { name, email, message }
  })
})

// File upload
app.post('/upload', async (c) => {
  const body = await c.req.parseBody()
  const file = body['file'] as File
  
  if (file) {
    return c.json({
      filename: file.name,
      size: file.size,
      type: file.type
    })
  }
  
  return c.text('No file uploaded', 400)
})
```

## Response Generation

### Different Response Types

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// Text response
app.get('/text', (c) => c.text('Hello World'))

// JSON response
app.get('/json', (c) => c.json({ message: 'Hello JSON' }))

// HTML response
app.get('/html', (c) => c.html('<h1>Hello HTML</h1>'))

// Custom response
app.get('/custom', (c) => {
  return c.newResponse('Custom content', 200, {
    'Content-Type': 'text/plain',
    'X-Custom-Header': 'custom-value'
  })
})

// Redirect
app.get('/redirect', (c) => c.redirect('/target'))
app.get('/target', (c) => c.text('Redirected successfully'))
```

### Status Codes and Headers

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.post('/users', async (c) => {
  const user = await c.req.json()
  
  // Set status code
  c.status(201)
  
  // Set headers
  c.header('Location', `/users/${user.id}`)
  c.header('X-Created-At', new Date().toISOString())
  
  return c.json({ message: 'User created', user })
})

// Multiple ways to set status
app.get('/status-examples', (c) => {
  // Method 1: Using status method
  c.status(418)
  return c.text("I'm a teapot")
  
  // Method 2: Inline with response
  // return c.text("I'm a teapot", 418)
  
  // Method 3: With headers
  // return c.text("I'm a teapot", 418, {
  //   'X-Teapot': 'true'
  // })
})
```

## Variable Management

### Setting and Getting Variables

```ts twoslash
import { Hono } from 'hono'

type Variables = {
  user: { id: string; name: string }
  requestId: string
  startTime: number
}

const app = new Hono<{ Variables: Variables }>()
// ---cut---
// Middleware to set variables
app.use('*', async (c, next) => {
  c.set('requestId', crypto.randomUUID())
  c.set('startTime', Date.now())
  await next()
})

// Auth middleware
app.use('/protected/*', async (c, next) => {
  // Simulate user authentication
  const token = c.req.header('Authorization')
  
  if (token) {
    const user = { id: '1', name: 'John Doe' } // From token
    c.set('user', user)
  }
  
  await next()
})

// Using variables in handlers
app.get('/protected/profile', (c) => {
  const user = c.get('user')
  const requestId = c.get('requestId')
  
  if (!user) {
    return c.text('Unauthorized', 401)
  }
  
  return c.json({ 
    profile: user,
    requestId
  })
})

// Performance timing
app.get('/timing', (c) => {
  const startTime = c.get('startTime')
  const duration = Date.now() - startTime
  
  return c.json({ 
    message: 'Request processed',
    duration: `${duration}ms`
  })
})
```

### Typed Variables

```ts twoslash
import { Hono } from 'hono'

interface User {
  id: string
  email: string
  role: 'admin' | 'user'
}

interface RequestContext {
  requestId: string
  user?: User
  isAuthenticated: boolean
  permissions: string[]
}

type Variables = RequestContext

const app = new Hono<{ Variables: Variables }>()
// ---cut---
// Type-safe variable access
app.use('*', async (c, next) => {
  c.set('requestId', crypto.randomUUID())
  c.set('isAuthenticated', false)
  c.set('permissions', [])
  await next()
})

app.use('/admin/*', async (c, next) => {
  const user = c.get('user')
  
  if (!user || user.role !== 'admin') {
    return c.text('Admin access required', 403)
  }
  
  c.set('permissions', ['read', 'write', 'delete'])
  await next()
})

app.get('/admin/users', (c) => {
  const permissions = c.get('permissions')
  const user = c.get('user')
  
  return c.json({
    adminUser: user?.email,
    permissions,
    message: 'Admin users list'
  })
})
```

## Environment Bindings

### Cloudflare Workers Bindings

```ts twoslash
import { Hono } from 'hono'

interface Bindings {
  DB: D1Database
  MY_KV: KVNamespace
  API_SECRET: string
  MY_BUCKET: R2Bucket
}

const app = new Hono<{ Bindings: Bindings }>()
// ---cut---
// Using KV storage
app.get('/config/:key', async (c) => {
  const key = c.req.param('key')
  const value = await c.env.MY_KV.get(key)
  
  if (!value) {
    return c.text('Config not found', 404)
  }
  
  return c.json({ key, value })
})

app.post('/config/:key', async (c) => {
  const key = c.req.param('key')
  const { value } = await c.req.json()
  
  await c.env.MY_KV.put(key, value)
  return c.json({ message: 'Config saved', key, value })
})

// Using D1 database
app.get('/users', async (c) => {
  const { results } = await c.env.DB.prepare(
    'SELECT id, email FROM users LIMIT 10'
  ).all()
  
  return c.json({ users: results })
})

// Using R2 bucket
app.post('/upload/:filename', async (c) => {
  const filename = c.req.param('filename')
  const file = await c.req.arrayBuffer()
  
  await c.env.MY_BUCKET.put(filename, file)
  return c.json({ message: 'File uploaded', filename })
})

// Using secrets
app.get('/external-api', async (c) => {
  const apiSecret = c.env.API_SECRET
  const response = await fetch('https://api.example.com/data', {
    headers: { 'Authorization': `Bearer ${apiSecret}` }
  })
  
  const data = await response.json()
  return c.json(data)
})

declare global {
  interface D1Database {
    prepare(sql: string): { all(): Promise<{ results: any[] }> }
  }
  
  interface KVNamespace {
    get(key: string): Promise<string | null>
    put(key: string, value: string): Promise<void>
  }
  
  interface R2Bucket {
    put(key: string, value: ArrayBuffer): Promise<void>
  }
}
```

### Node.js Environment Variables

```ts twoslash
import { Hono } from 'hono'

interface NodeBindings {
  DATABASE_URL: string
  JWT_SECRET: string
  REDIS_URL: string
  PORT: string
}

const app = new Hono<{ Bindings: NodeBindings }>()

// In Node.js, you'd typically pass process.env as bindings
// serve(app, { env: process.env })
// ---cut---
app.get('/db-info', (c) => {
  const dbUrl = c.env.DATABASE_URL
  
  // Don't expose full URL in response
  const dbHost = new URL(dbUrl).hostname
  
  return c.json({
    database: 'connected',
    host: dbHost,
    status: 'active'
  })
})

app.get('/health', (c) => {
  const port = c.env.PORT
  
  return c.json({
    status: 'healthy',
    port: parseInt(port),
    timestamp: new Date().toISOString()
  })
})
```

## Rendering and Templating

### HTML Rendering

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/profile/:name', (c) => {
  const name = c.req.param('name')
  
  const html = `
    <!DOCTYPE html>
    <html>
    <head>
      <title>Profile - ${name}</title>
    </head>
    <body>
      <h1>Welcome, ${name}!</h1>
      <p>This is your profile page.</p>
    </body>
    </html>
  `
  
  return c.html(html)
})
```

### JSON Responses with Custom Headers

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/api/data', (c) => {
  const data = {
    items: ['item1', 'item2', 'item3'],
    count: 3,
    timestamp: new Date().toISOString()
  }
  
  // Add CORS headers
  c.header('Access-Control-Allow-Origin', '*')
  c.header('Cache-Control', 'max-age=300')
  
  return c.json(data)
})

// Pretty JSON in development
app.get('/api/debug', (c) => {
  const debugInfo = {
    environment: 'development',
    version: '1.0.0',
    features: ['auth', 'uploads', 'notifications']
  }
  
  return c.json(debugInfo, 200, {
    'Content-Type': 'application/json; charset=utf-8'
  })
})
```

## Error Handling with Context

### Custom Error Responses

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/users/:id', async (c) => {
  const id = c.req.param('id')
  
  if (!id || isNaN(parseInt(id))) {
    c.status(400)
    return c.json({
      error: 'Invalid user ID',
      code: 'INVALID_ID',
      path: c.req.path
    })
  }
  
  // Simulate user lookup
  const user = await findUser(id)
  
  if (!user) {
    c.status(404)
    return c.json({
      error: 'User not found',
      code: 'USER_NOT_FOUND',
      userId: id
    })
  }
  
  return c.json(user)
})

declare const findUser: (id: string) => Promise<any>
```

### Error Context Information

```ts twoslash
import { Hono } from 'hono'

type Variables = {
  requestId: string
}

const app = new Hono<{ Variables: Variables }>()
// ---cut---
app.onError((err, c) => {
  const errorInfo = {
    error: err.message,
    path: c.req.path,
    method: c.req.method,
    timestamp: new Date().toISOString(),
    userAgent: c.req.header('user-agent'),
    requestId: c.get('requestId') || 'unknown'
  }
  
  console.error('Application error:', errorInfo)
  
  return c.json({
    error: 'Internal Server Error',
    requestId: errorInfo.requestId
  }, 500)
})
```

## Authentication Patterns

### JWT Authentication

```ts twoslash
import { Hono } from 'hono'

interface JWTPayload {
  userId: string
  email: string
  role: string
  exp: number
}

type Variables = {
  user: JWTPayload
  isAuthenticated: boolean
}

const app = new Hono<{ Variables: Variables }>()

declare const verifyJWT: (token: string) => Promise<JWTPayload>
// ---cut---
// JWT middleware
app.use('/protected/*', async (c, next) => {
  const authHeader = c.req.header('Authorization')
  
  if (!authHeader?.startsWith('Bearer ')) {
    c.set('isAuthenticated', false)
    return c.json({ error: 'Authorization required' }, 401)
  }
  
  const token = authHeader.slice(7)
  
  try {
    const payload = await verifyJWT(token)
    
    if (payload.exp < Date.now() / 1000) {
      return c.json({ error: 'Token expired' }, 401)
    }
    
    c.set('user', payload)
    c.set('isAuthenticated', true)
  } catch (error) {
    return c.json({ error: 'Invalid token' }, 401)
  }
  
  await next()
})

app.get('/protected/me', (c) => {
  const user = c.get('user')
  return c.json({ user })
})

app.get('/protected/admin', async (c, next) => {
  const user = c.get('user')
  
  if (user.role !== 'admin') {
    return c.json({ error: 'Admin access required' }, 403)
  }
  
  await next()
}, (c) => {
  return c.json({ message: 'Admin area', user: c.get('user') })
})
```

### Session-Based Authentication

```ts twoslash
import { Hono } from 'hono'

interface Session {
  userId: string
  email: string
  createdAt: number
}

type Variables = {
  session: Session | null
  userId: string | null
}

const app = new Hono<{ Variables: Variables }>()

// Simplified session store (use Redis in production)
const sessions = new Map<string, Session>()

declare const getCookie: (c: any, name: string) => string | undefined
declare const setCookie: (c: any, name: string, value: string, options?: any) => void
// ---cut---
// Session middleware
app.use('*', async (c, next) => {
  const sessionId = getCookie(c, 'session_id')
  
  if (sessionId) {
    const session = sessions.get(sessionId)
    if (session) {
      c.set('session', session)
      c.set('userId', session.userId)
    } else {
      c.set('session', null)
      c.set('userId', null)
    }
  } else {
    c.set('session', null)
    c.set('userId', null)
  }
  
  await next()
})

app.post('/login', async (c) => {
  const { email, password } = await c.req.json()
  
  // Validate credentials (simplified)
  if (email === 'user@example.com' && password === 'password') {
    const sessionId = crypto.randomUUID()
    const session: Session = {
      userId: '1',
      email,
      createdAt: Date.now()
    }
    
    sessions.set(sessionId, session)
    
    setCookie(c, 'session_id', sessionId, {
      httpOnly: true,
      secure: true,
      maxAge: 86400 // 24 hours
    })
    
    return c.json({ message: 'Logged in successfully' })
  }
  
  return c.json({ error: 'Invalid credentials' }, 401)
})

app.get('/profile', (c) => {
  const session = c.get('session')
  
  if (!session) {
    return c.json({ error: 'Not authenticated' }, 401)
  }
  
  return c.json({
    user: {
      id: session.userId,
      email: session.email
    }
  })
})
```

## Platform-Specific Features

### Cloudflare Workers Specific

```ts
import { Hono } from 'hono'

interface CFBindings {
  MY_KV: any // Simplified for example
  ANALYTICS: any
}

const app = new Hono<{ Bindings: CFBindings }>()

app.get('/cf-info', (c) => {
  // Access Cloudflare request properties
  const cf = c.req.raw.cf
  
  return c.json({
    country: cf?.country,
    city: cf?.city,
    region: cf?.region,
    timezone: cf?.timezone,
    colo: cf?.colo,
    httpProtocol: cf?.httpProtocol,
    tlsVersion: cf?.tlsVersion
  })
})

app.get('/analytics/:event', async (c) => {
  const event = c.req.param('event')
  
  // Log to Cloudflare Analytics
  c.env.ANALYTICS.writeDataPoint({
    blobs: ['event', event],
    doubles: [Date.now()],
    indexes: [event]
  })
  
  return c.json({ message: 'Event logged', event })
})
```
```

### Node.js Specific Features

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/server-info', (c) => {
  // Node.js-specific information
  return c.json({
    nodeVersion: process.version,
    platform: process.platform,
    arch: process.arch,
    uptime: process.uptime(),
    memoryUsage: process.memoryUsage(),
    environment: process.env.NODE_ENV
  })
})

app.get('/request-details', (c) => {
  // Additional request information available in Node.js
  return c.json({
    method: c.req.method,
    url: c.req.url,
    headers: Object.fromEntries(
      Object.entries(c.req.raw.headers).map(([k, v]) => [k.toLowerCase(), v])
    ),
    timestamp: new Date().toISOString()
  })
})
```

## Testing Context Usage

### Testing Variables and Environment

```ts twoslash
import { Hono } from 'hono'

declare const test: (name: string, fn: () => void) => void
declare const expect: (value: any) => any

interface TestBindings {
  TEST_SECRET: string
}

type TestVariables = {
  testData: string
}

const app = new Hono<{ 
  Bindings: TestBindings
  Variables: TestVariables 
}>()

app.use('*', (c, next) => {
  c.set('testData', 'test-value')
  return next()
})

app.get('/test-endpoint', (c) => {
  const testData = c.get('testData')
  const secret = c.env.TEST_SECRET
  
  return c.json({ testData, hasSecret: !!secret })
})
// ---cut---
test('context variables work', async () => {
  const res = await app.request('/test-endpoint', {}, {
    TEST_SECRET: 'test-secret-value'
  })
  
  const result = await res.json()
  
  expect(result.testData).toBe('test-value')
  expect(result.hasSecret).toBe(true)
})

test('context status and headers', async () => {
  const testApp = new Hono()
  
  testApp.get('/status-test', (c) => {
    c.status(201)
    c.header('X-Test', 'success')
    return c.json({ message: 'created' })
  })
  
  const res = await testApp.request('/status-test')
  
  expect(res.status).toBe(201)
  expect(res.headers.get('X-Test')).toBe('success')
  expect(await res.json()).toEqual({ message: 'created' })
})
```

## See Also

- [Context Specification](/docs/api/context) - Complete Context API reference
- [HonoRequest Examples](/docs/api/request-examples) - Request handling patterns
- [Hono Examples](/docs/api/hono-examples) - Application setup and configuration
- [Middleware Guide](/docs/guides/middleware) - Context usage in middleware