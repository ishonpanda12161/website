# Router Preset Examples

Practical examples demonstrating how to choose and use different Hono router presets for various deployment scenarios.

## Basic Usage

### Default Preset (`hono`)

```ts twoslash
import { Hono } from 'hono'

const app = new Hono()

// All routing features available
app.get('/', (c) => c.text('Hello World'))
app.get('/users/:id', (c) => c.text(`User: ${c.req.param('id')}`))
app.get('/posts/:id{[0-9]+}', (c) => c.text(`Post: ${c.req.param('id')}`))
app.get('/files/*', (c) => c.text('File path'))

export default app
```

### Quick Preset (`hono/quick`)

```ts twoslash
import { Hono } from 'hono/quick'

const app = new Hono()

// Fast initialization for per-request environments
app.get('/', (c) => c.text('Hello Quick'))
app.get('/users/:id', (c) => c.text(`User: ${c.req.param('id')}`))
app.get('/api/*', (c) => c.text('API endpoint'))

export default app
```

### Tiny Preset (`hono/tiny`)

```ts twoslash
import { Hono } from 'hono/tiny'

const app = new Hono()

// Minimal footprint for resource-constrained environments
app.get('/', (c) => c.text('Hello Tiny'))
app.get('/health', (c) => c.json({ status: 'ok' }))
app.get('/users/:id', (c) => c.text(`User: ${c.req.param('id')}`))

export default app
```

## Platform-Specific Examples

### Cloudflare Workers (Default Preset)

```ts
import { Hono } from 'hono'

const app = new Hono()

app.get('/', (c) => c.text('Hello Cloudflare Workers'))

// Complex routing patterns work well
app.get('/api/v1/users/:id', (c) => {
  const id = c.req.param('id')
  // Validate numeric ID in handler
  if (!/^\d+$/.test(id)) {
    return c.text('Invalid user ID', 400)
  }
  return c.json({ userId: parseInt(id) })
})

app.get('/files/:filename', (c) => {
  const filename = c.req.param('filename')
  // Check file extension in handler
  if (!/\.(jpg|png|gif)$/i.test(filename)) {
    return c.text('Invalid file type', 400)
  }
  return c.text(`Image file: ${filename}`)
})

// Cloudflare Workers export
export default app
```

### Fastly Compute (Quick Preset)

```ts twoslash
import { Hono } from 'hono/quick'

const app = new Hono()

// Optimized for per-request initialization
app.get('/', (c) => c.text('Hello Fastly Compute'))

app.get('/cache/:key', async (c) => {
  const key = c.req.param('key')
  // Fastly-specific caching logic
  return c.text(`Cached value for: ${key}`)
})

// Simple patterns work well
app.get('/api/:version/:resource', (c) => {
  const { version, resource } = c.req.param()
  return c.json({ version, resource })
})

// Fastly Compute export
export default app
```

### AWS Lambda (Quick Preset)

```ts twoslash
import { Hono } from 'hono/quick'
import { handle } from 'hono/aws-lambda'

const app = new Hono()

// Fast cold start initialization
app.get('/', (c) => c.json({ message: 'Hello Lambda' }))

app.post('/webhook', async (c) => {
  const body = await c.req.json()
  console.log('Webhook received:', body)
  return c.json({ received: true })
})

app.get('/users/:id', async (c) => {
  const id = c.req.param('id')
  // Database lookup
  return c.json({ userId: id })
})

// Lambda handler
export const handler = handle(app)
```

### Node.js Server (Default Preset)

```ts
import { Hono } from 'hono'
import { serve } from '@hono/node-server'

const app = new Hono()

// Long-running server benefits from default preset
app.get('/', (c) => c.text('Hello Node.js'))

// Complex routing patterns with manual validation
app.get('/api/posts/:year/:month/:slug', (c) => {
  const { year, month, slug } = c.req.param()
  
  // Manual validation instead of regex constraints
  if (!/^\d{4}$/.test(year) || !/^\d{2}$/.test(month) || !/^[a-z0-9-]+$/.test(slug)) {
    return c.text('Invalid parameters', 400)
  }
  
  return c.json({ 
    post: { year: parseInt(year), month: parseInt(month), slug }
  })
})

// Simple file serving pattern
app.get('/static/:file', (c) => {
  const file = c.req.param('file')
  
  // Validate file extension in handler
  if (!/\.(css|js|png|jpg|svg)$/.test(file)) {
    return c.text('Invalid file type', 400)
  }
  
  return c.text(`Serving: ${file}`)
})

// Start server
const port = 3000
serve(app, (info) => {
  console.log(`Listening on http://localhost:${info.port}`)
})
```

### IoT/Embedded (Tiny Preset)

```ts twoslash
import { Hono } from 'hono/tiny'

// Minimal application for resource-constrained environment
const app = new Hono()

// Simple health check
app.get('/health', (c) => c.json({ status: 'ok', uptime: process.uptime() }))

// Basic sensor data endpoint
app.get('/sensors/:id', (c) => {
  const id = c.req.param('id')
  return c.json({ 
    sensorId: id,
    value: Math.random() * 100,
    timestamp: Date.now()
  })
})

// Configuration endpoint
app.post('/config', async (c) => {
  const config = await c.req.json()
  return c.json({ message: 'Config updated', config })
})

export default app
```

## Performance Comparison Examples

### Route Registration Performance

```ts twoslash
// Default preset - slower initialization, better runtime
import { Hono } from 'hono'

const app = new Hono()

console.time('route registration')

// Register 1000 routes
for (let i = 0; i < 1000; i++) {
  app.get(`/route-${i}`, (c) => c.text(`Route ${i}`))
  app.get(`/users/:id/posts/${i}`, (c) => c.text(`User post ${i}`))
  app.get(`/files/:name{.+\\.${i % 10}}`, (c) => c.text(`File ${i}`))
}

console.timeEnd('route registration')
// Output: Slower registration but excellent runtime performance
```

```ts twoslash
// Quick preset - faster initialization
import { Hono } from 'hono/quick'

const app = new Hono()

console.time('quick route registration')

// Same 1000 routes
for (let i = 0; i < 1000; i++) {
  app.get(`/route-${i}`, (c) => c.text(`Route ${i}`))
  app.get(`/users/:id/posts/${i}`, (c) => c.text(`User post ${i}`))
  // Complex regex patterns may be limited
  app.get(`/files/:name`, (c) => c.text(`File ${i}`))
}

console.timeEnd('quick route registration')
// Output: Faster registration, good runtime performance
```

```ts twoslash
// Tiny preset - fastest initialization
import { Hono } from 'hono/tiny'

const app = new Hono()

console.time('tiny route registration')

// Simplified routes for tiny preset
for (let i = 0; i < 1000; i++) {
  app.get(`/route-${i}`, (c) => c.text(`Route ${i}`))
  app.get(`/users/:id/posts/${i}`, (c) => c.text(`User post ${i}`))
  // No regex constraints supported
}

console.timeEnd('tiny route registration')
// Output: Fastest registration, basic runtime performance
```

## Feature Support Examples

### Complex Routing (Default Preset Only)

```ts twoslash
import { Hono } from 'hono'

const app = new Hono()

// Advanced regex patterns - only supported in default preset
app.get('/posts/:date{[0-9]{4}-[0-9]{2}-[0-9]{2}}/:slug{[a-z0-9-]+}', (c) => {
  const { date, slug } = c.req.param()
  return c.json({ date, slug })
})

// Complex file extension matching
app.get('/assets/:file{.+\\.(js|css|png|jpg|gif|svg|woff2?)}', (c) => {
  const file = c.req.param('file')
  const ext = file.split('.').pop()
  return c.text(`Serving ${ext} file: ${file}`)
})

// Email-like pattern matching
app.get('/users/:email{[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}}', (c) => {
  const email = c.req.param('email')
  return c.json({ userEmail: email })
})
```

### Simple Patterns (All Presets)

```ts twoslash
// This works with all presets
import { Hono } from 'hono' // or 'hono/quick' or 'hono/tiny'

const app = new Hono()

// Basic parameters
app.get('/users/:id', (c) => c.text(`User ${c.req.param('id')}`))

// Optional parameters
app.get('/posts/:id/:slug?', (c) => {
  const { id, slug } = c.req.param()
  return c.json({ id, slug: slug || 'no-slug' })
})

// Wildcards
app.get('/files/*', (c) => c.text('File handler'))
app.get('/api/v1/*', (c) => c.text('API v1'))

// Multiple parameters
app.get('/users/:userId/posts/:postId', (c) => {
  const { userId, postId } = c.req.param()
  return c.json({ userId, postId })
})
```

## Migration Examples

### Migrating from Default to Quick

```ts twoslash
// Before: Using default preset with complex patterns
// import { Hono } from 'hono'

// After: Using quick preset - remove complex regex
import { Hono } from 'hono/quick'

const app = new Hono()

// ❌ This won't work well in quick preset
// app.get('/posts/:date{[0-9]{4}-[0-9]{2}-[0-9]{2}}', handler)

// ✅ Use simpler patterns instead
app.get('/posts/:year/:month/:day', (c) => {
  const { year, month, day } = c.req.param()
  
  // Validate in handler instead of regex
  if (!/^\d{4}$/.test(year) || !/^\d{2}$/.test(month) || !/^\d{2}$/.test(day)) {
    return c.text('Invalid date format', 400)
  }
  
  return c.json({ year, month, day })
})
```

### Migrating from Quick to Tiny

```ts twoslash
// Before: Using quick preset
// import { Hono } from 'hono/quick'

// After: Using tiny preset - simplify further
import { Hono } from 'hono/tiny'

const app = new Hono()

// Keep simple routes
app.get('/', (c) => c.text('Home'))
app.get('/health', (c) => c.json({ status: 'ok' }))

// Simple parameter routes work fine
app.get('/users/:id', (c) => {
  const id = c.req.param('id')
  return c.json({ userId: id })
})

// Wildcard routes work
app.get('/api/*', (c) => c.text('API endpoint'))
```

## Development vs Production Examples

### Development Setup (Any Preset)

```ts
import { Hono } from 'hono' // Use default for full feature support during development

const app = new Hono()

// Development-friendly error handling
app.use('*', async (c, next) => {
  try {
    await next()
  } catch (error) {
    const err = error as Error
    console.error('Development error:', err)
    return c.json({ 
      error: err.message,
      stack: err.stack 
    }, 500)
  }
})

// Complex routes for testing all features  
app.get('/debug/:param/:num', (c) => {
  const { param, num } = c.req.param()
  
  // Manual validation instead of regex
  if (!/^[a-z]+$/.test(param) || !/^[0-9]+$/.test(num)) {
    return c.text('Invalid parameters', 400)
  }
  
  return c.json({ param, num: parseInt(num), env: 'development' })
})

export default app
```

### Production Optimization

```ts
// Choose preset based on deployment platform
import { Hono } from 'hono/quick' // Example: Using quick for Lambda

const app = new Hono()

// Production error handling
app.use('*', async (c, next) => {
  try {
    await next()
  } catch (error) {
    const err = error as Error
    console.error('Production error:', err.message)
    return c.json({ error: 'Internal Server Error' }, 500)
  }
})

// Optimized routes for chosen preset
app.get('/api/users/:id', (c) => {
  const id = c.req.param('id')
  
  // Input validation in handler for quick/tiny presets
  if (!/^\d+$/.test(id)) {
    return c.json({ error: 'Invalid user ID' }, 400)
  }
  
  return c.json({ userId: parseInt(id) })
})

export default app
```

## Bundle Size Comparison

### Default Preset Application

```ts twoslash
import { Hono } from 'hono'
import { cors } from 'hono/cors'
import { logger } from 'hono/logger'

const app = new Hono()

app.use('*', cors())
app.use('*', logger())

// Full feature set available
app.get('/complex/:pattern{[a-z0-9-]+}/:id{[0-9]+}', (c) => {
  return c.json({ 
    pattern: c.req.param('pattern'),
    id: parseInt(c.req.param('id'))
  })
})

export default app
// Bundle size: ~50-60KB (gzipped)
```

### Tiny Preset Application

```ts twoslash
import { Hono } from 'hono/tiny'

const app = new Hono()

// Minimal middleware and features
app.get('/simple/:id', (c) => {
  return c.json({ id: c.req.param('id') })
})

export default app
// Bundle size: ~15-20KB (gzipped)
```

## Testing Across Presets

### Preset-Agnostic Tests

```ts twoslash
import { Hono } from 'hono' // Switch this import to test different presets

declare const test: (name: string, fn: () => void) => void
declare const expect: (value: any) => any

const createApp = () => {
  const app = new Hono()
  
  app.get('/', (c) => c.text('Hello'))
  app.get('/users/:id', (c) => c.text(`User: ${c.req.param('id')}`))
  app.post('/data', async (c) => {
    const body = await c.req.json()
    return c.json({ received: body })
  })
  
  return app
}

// These tests work with any preset
test('basic routes work', async () => {
  const app = createApp()
  
  const res = await app.request('/')
  expect(res.status).toBe(200)
  expect(await res.text()).toBe('Hello')
})

test('parameter routes work', async () => {
  const app = createApp()
  
  const res = await app.request('/users/123')
  expect(res.status).toBe(200)
  expect(await res.text()).toBe('User: 123')
})
```

## See Also

- [Router Preset Specification](/docs/api/presets) - Complete preset reference
- [Performance Benchmarks](/docs/concepts/benchmarks) - Detailed performance comparisons
- [Routing Examples](/docs/api/routing-examples) - Route pattern examples
- [Hono Examples](/docs/api/hono-examples) - Application setup patterns