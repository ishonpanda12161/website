# Context Examples

Practical examples and usage patterns for the Hono Context object.

## Basic Usage Examples

### Request Information Access

```ts
app.get('/debug', (c) => {
  // Access path parameters
  const id = c.req.param('id')
  
  // Access query string  
  const query = c.req.query('q')
  
  // Parse request body
  const body = await c.req.parseBody()
  
  return c.json({ id, query, body })
})
```

### Response Methods

```ts
app.get('/responses', (c) => {
  // Text response
  if (c.req.query('format') === 'text') {
    return c.text('Hello world', 200, { 'X-Custom-Header': 'value' })
  }
  
  // JSON response
  if (c.req.query('format') === 'json') {
    return c.json({ message: 'Success', data: { id: 123 } })
  }
  
  // HTML response
  if (c.req.query('format') === 'html') {
    return c.html('<h1>Hello world</h1>')
  }
  
  // Default response
  return c.text('Use ?format=text|json|html')
})
```

## Environment Bindings Examples

### Cloudflare Workers

```ts
type Env = {
  MY_KV: KVNamespace
  API_KEY: string
  DB: D1Database
}

const app = new Hono<{ Bindings: Env }>()

app.get('/kv/:key', async (c) => {
  const key = c.req.param('key')
  
  // Access KV namespace
  const value = await c.env.MY_KV.get(key)
  
  if (!value) {
    return c.json({ error: 'Key not found' }, 404)
  }
  
  return c.json({ key, value })
})

app.get('/config', (c) => {
  // Access secret
  const apiKey = c.env.API_KEY
  
  return c.json({ 
    hasApiKey: !!apiKey,
    keyLength: apiKey?.length 
  })
})
```

### Database Integration

```ts
app.get('/users/:id', async (c) => {
  const id = c.req.param('id')
  
  try {
    // Access D1 database from environment
    const result = await c.env.DB.prepare(
      'SELECT * FROM users WHERE id = ?'
    ).bind(id).first()
    
    if (!result) {
      return c.json({ error: 'User not found' }, 404)
    }
    
    return c.json(result)
  } catch (error) {
    return c.json({ error: 'Database error' }, 500)
  }
})
```

## Variable Management Examples

### Setting Variables in Middleware

```ts
import { createMiddleware } from 'hono/factory'

type Variables = {
  user: { id: string; name: string }
  requestId: string
}

const authMiddleware = createMiddleware<{ Variables: Variables }>(
  async (c, next) => {
    // Generate request ID
    c.set('requestId', crypto.randomUUID())
    
    // Authenticate user
    const token = c.req.header('Authorization')?.replace('Bearer ', '')
    if (token) {
      const user = await verifyToken(token)
      c.set('user', user)
    }
    
    await next()
  }
)

app.use('*', authMiddleware)

app.get('/profile', (c) => {
  const user = c.get('user')
  const requestId = c.get('requestId')
  
  if (!user) {
    return c.json({ error: 'Unauthorized' }, 401)
  }
  
  return c.json({ user, requestId })
})
```

### Type-Safe Variable Access

```ts
app.get('/dashboard', (c) => {
  // Type-safe variable access
  const user = c.var.user        // Type: { id: string; name: string }
  const requestId = c.var.requestId  // Type: string
  
  return c.html(`
    <h1>Welcome ${user.name}</h1>
    <p>Request ID: ${requestId}</p>
  `)
})
```

## Response Header Examples

### Method Chaining

```ts
app.get('/api/data', (c) => {
  return c
    .status(201)
    .header('X-Custom', 'value')
    .header('Cache-Control', 'no-cache')
    .json({ created: true })
})
```

### CORS Headers

```ts
app.get('/api/public', (c) => {
  return c
    .header('Access-Control-Allow-Origin', '*')
    .header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE')
    .header('Access-Control-Allow-Headers', 'Content-Type, Authorization')
    .json({ message: 'Public API endpoint' })
})
```

## Custom Response Examples

### File Downloads

```ts
app.get('/download/:filename', async (c) => {
  const filename = c.req.param('filename')
  
  // Get file data (example)
  const fileData = await getFileData(filename)
  
  return c.body(fileData, 200, {
    'Content-Type': 'application/octet-stream',
    'Content-Disposition': `attachment; filename="${filename}"`
  })
})
```

### Streaming Responses

```ts
app.get('/stream', (c) => {
  const stream = new ReadableStream({
    start(controller) {
      // Send data chunks
      controller.enqueue(new TextEncoder().encode('chunk 1\n'))
      controller.enqueue(new TextEncoder().encode('chunk 2\n'))
      controller.close()
    }
  })
  
  return c.body(stream, 200, {
    'Content-Type': 'text/plain',
    'Transfer-Encoding': 'chunked'
  })
})
```

## Redirect Examples

### Simple Redirects

```ts
app.get('/old-page', (c) => {
  return c.redirect('/new-page')  // 302 temporary redirect
})

app.get('/moved-permanently', (c) => {
  return c.redirect('/new-location', 301)  // 301 permanent redirect
})
```

### Conditional Redirects

```ts
app.get('/dashboard', (c) => {
  const user = c.var.user
  
  if (!user) {
    return c.redirect('/login')
  }
  
  if (!user.emailVerified) {
    return c.redirect('/verify-email')
  }
  
  return c.html('<h1>Dashboard</h1>')
})
```

## Error Handling Examples

### Custom 404 Pages

```ts
app.notFound((c) => {
  if (c.req.header('Accept')?.includes('application/json')) {
    return c.json({ error: 'Not found' }, 404)
  }
  
  return c.html(`
    <html>
      <body>
        <h1>Page Not Found</h1>
        <p>The page ${c.req.path} was not found.</p>
      </body>
    </html>
  `, 404)
})
```

### Request ID Logging

```ts
app.use('*', async (c, next) => {
  const requestId = crypto.randomUUID()
  c.set('requestId', requestId)
  
  console.log(`[${requestId}] ${c.req.method} ${c.req.path}`)
  
  await next()
  
  console.log(`[${requestId}] Response sent`)
})
```

## Rendering Examples

### Template Rendering

```ts
import { createMiddleware } from 'hono/factory'

// Set up renderer
const renderer = createMiddleware(async (c, next) => {
  c.setRenderer((content, props) => {
    return c.html(`
      <!DOCTYPE html>
      <html>
        <head><title>${props?.title || 'App'}</title></head>
        <body>
          <header><h1>My App</h1></header>
          <main>${content}</main>
        </body>
      </html>
    `)
  })
  await next()
})

app.use('*', renderer)

app.get('/page', (c) => {
  return c.render('<p>Page content</p>', { title: 'My Page' })
})
```

### JSX Rendering

```tsx
import { jsx } from 'hono/jsx'

const Layout = ({ children, title }: { children: any; title: string }) => (
  <html>
    <head><title>{title}</title></head>
    <body>{children}</body>
  </html>
)

app.get('/jsx', (c) => {
  return c.html(
    <Layout title="JSX Example">
      <h1>Hello from JSX!</h1>
      <p>User: {c.var.user?.name || 'Guest'}</p>
    </Layout>
  )
})
```

## Platform-Specific Examples

### Cloudflare Workers ExecutionContext

```ts
app.post('/background-task', async (c) => {
  // Start background task
  c.executionCtx.waitUntil(
    fetch('https://api.example.com/log', {
      method: 'POST',
      body: JSON.stringify({ action: 'background-task' })
    })
  )
  
  return c.json({ message: 'Task started' })
})
```

### Custom Response with Platform Features

```ts
app.get('/edge-info', (c) => {
  const cf = c.req.raw.cf  // Cloudflare-specific properties
  
  return c.json({
    colo: cf?.colo,
    country: cf?.country,
    city: cf?.city,
    asn: cf?.asn
  })
})
```

## Testing Examples

### Mock Context for Testing

```ts
import { Context } from 'hono'

// Testing helper
const createTestContext = (path: string, options: any = {}) => {
  const req = new Request(`http://localhost${path}`, options)
  return new Context(req)
}

// Test example
describe('Handler tests', () => {
  test('should return user data', async () => {
    const c = createTestContext('/users/123')
    c.set('user', { id: '123', name: 'Test User' })
    
    const response = await userHandler(c)
    const data = await response.json()
    
    expect(data.user.name).toBe('Test User')
  })
})
```

## See Also

- [Context Specification](/docs/api/context) - Complete formal specification
- [HonoRequest Examples](/docs/api/request-examples) - Request handling patterns
- [Middleware Guide](/docs/guides/middleware) - Context usage in middleware