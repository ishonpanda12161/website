# Router Preset Examples

Practical examples and usage patterns for different Hono router presets.

## Basic Usage Examples

### Default Preset (`hono`)

```ts twoslash
import { Hono } from 'hono'
// ---cut---
const app = new Hono()

// This uses SmartRouter with RegExpRouter and TrieRouter
app.get('/users/:id', (c) => c.json({ id: c.req.param('id') }))
app.get('/posts/*', (c) => c.text('Posts section'))
```

### Quick Preset (`hono/quick`)

```ts twoslash
import { Hono } from 'hono/quick'
// ---cut---
const app = new Hono()

// This uses SmartRouter with LinearRouter and TrieRouter
app.get('/api/data', (c) => c.json({ data: 'quick response' }))
app.post('/api/webhook', (c) => c.text('OK'))
```

### Tiny Preset (`hono/tiny`)

```ts twoslash
import { Hono } from 'hono/tiny'
// ---cut---
const app = new Hono()

// This uses PatternRouter - smallest bundle size
app.get('/health', (c) => c.text('OK'))
app.post('/ping', (c) => c.text('pong'))
```

## Platform-Specific Usage

### Long-Running Servers (Node.js, Deno, Bun)

```ts
import { Hono } from 'hono'
const app = new Hono()

// Complex routing patterns work well with default preset
app.get('/users/:id', (c) => {
  return c.json({ userId: c.req.param('id') })
})

app.get('/posts/:slug', (c) => {
  return c.json({ postSlug: c.req.param('slug') })
})

// Multiple complex routes benefit from RegExpRouter optimization
for (let i = 0; i < 100; i++) {
  app.get(`/api/v1/resource${i}/:id`, (c) => {
    return c.json({ resource: i, id: c.req.param('id') })
  })
}
```

// Deno
if (typeof Deno !== 'undefined') {
  Deno.serve(app.fetch)
}

// Node.js  
if (typeof process !== 'undefined' && process.versions?.node) {
  const { serve } = await import('@hono/node-server')
  serve(app, (info) => console.log(`Server running at ${info.port}`))
}

// Bun
export default {
  port: 3000,
  fetch: app.fetch,
}
```

### Fastly Compute@Edge

```ts twoslash
import { Hono } from 'hono/quick'
// ---cut---
// Use quick preset for per-request initialization
const app = new Hono()

app.get('/edge/:region', (c) => {
  const region = c.req.param('region')
  return c.json({ 
    region,
    edge: 'fastly',
    timestamp: Date.now()
  })
})

app.post('/cache-invalidate', async (c) => {
  // Fast initialization is important here
  return c.json({ status: 'invalidated' })
})

addEventListener('fetch', (event) => {
  event.respondWith(app.fetch(event.request))
})
```

### Cloudflare Workers

```ts  
import { Hono } from 'hono'

// Default preset works well due to isolate persistence
const app = new Hono()

app.get('/cf/:zone', (c) => {
  const raw = c.req.raw as Request & { cf?: any }
  return c.json({
    zone: c.req.param('zone'),
    cf: raw.cf,
    colo: raw.cf?.colo
  })
})

export default app
```

### Resource-Constrained Environments

```ts twoslash
import { Hono } from 'hono/tiny'
// ---cut---
// Minimal bundle size for IoT or embedded environments
const app = new Hono()

// Simple routes work best with PatternRouter
app.get('/', (c) => c.text('IoT Device API'))
app.post('/data', (c) => c.json({ received: true }))
app.get('/status', (c) => c.json({ status: 'online', memory: 'low' }))

export default app
```

## Performance Comparison Examples

### Route Registration Performance

```ts
// Default preset - slower registration, faster execution
import { Hono as DefaultHono } from 'hono'

// Quick preset - faster registration, good execution
import { Hono as QuickHono } from 'hono/quick'

// Tiny preset - fastest registration, simple execution
import { Hono as TinyHono } from 'hono/tiny'

const testRoutes = [
  '/api/users/:id',
  '/api/posts/:slug',
  '/api/comments/:postId/:commentId',
  '/health',
  '/metrics',
  '/docs/*'
]

// Benchmark route registration
console.time('Default registration')
const defaultApp = new DefaultHono()
testRoutes.forEach(route => defaultApp.get(route, (c) => c.text('OK')))
console.timeEnd('Default registration')

console.time('Quick registration')  
const quickApp = new QuickHono()
testRoutes.forEach(route => quickApp.get(route, (c) => c.text('OK')))
console.timeEnd('Quick registration')

console.time('Tiny registration')
const tinyApp = new TinyHono()
testRoutes.forEach(route => tinyApp.get(route, (c) => c.text('OK')))
console.timeEnd('Tiny registration')
```

### Bundle Size Comparison

```ts
// Bundle size impact (approximate)

// hono - ~35KB minified
import { Hono } from 'hono'

// hono/quick - ~30KB minified  
import { Hono } from 'hono/quick'

// hono/tiny - ~15KB minified
import { Hono } from 'hono/tiny'
```

## Migration Examples

### From Default to Quick

```ts
// Before (using default preset)
import { Hono } from 'hono'

// After (using quick preset)
import { Hono } from 'hono/quick'

// No other changes needed - API is identical
const app = new Hono()
app.get('/', (c) => c.text('Hello'))
```

### From Default to Tiny

```ts
// Before (using default preset with complex patterns)
import { Hono } from 'hono'
const app = new Hono()

app.get('/users/:id{[0-9]+}', handler)    // Complex regex
app.get('/posts/*', handler)               // Wildcard

// After (simplified for tiny preset)
import { Hono } from 'hono/tiny'
const app = new Hono()

app.get('/users/:id', handler)    // Simple parameter
app.get('/posts/:path', handler)  // Named parameter instead of wildcard
```

## Testing Different Presets

```ts
// Test compatibility across presets
const presets = [
  { name: 'default', Hono: (await import('hono')).Hono },
  { name: 'quick', Hono: (await import('hono/quick')).Hono },
  { name: 'tiny', Hono: (await import('hono/tiny')).Hono }
]

for (const preset of presets) {
  console.log(`Testing ${preset.name} preset`)
  
  const app = new preset.Hono()
  app.get('/test/:id', (c) => c.json({ id: c.req.param('id') }))
  
  const response = await app.request('/test/123')
  const result = await response.json()
  
  console.log(`${preset.name}: ${JSON.stringify(result)}`)
}
```

## Environment Detection

```ts
// Auto-select preset based on environment
let Hono

if (typeof process !== 'undefined' && process.env.NODE_ENV === 'production') {
  // Long-running server - use default
  Hono = (await import('hono')).Hono
} else if (typeof EdgeRuntime !== 'undefined') {
  // Edge runtime - use quick  
  Hono = (await import('hono/quick')).Hono
} else if (globalThis.RESOURCE_CONSTRAINED) {
  // Resource constrained - use tiny
  Hono = (await import('hono/tiny')).Hono
} else {
  // Development or unknown - use default
  Hono = (await import('hono')).Hono
}

const app = new Hono()
```

## Custom Router Configuration

```ts
// If presets don't fit, use custom router
import { Hono } from 'hono'
import { RegExpRouter } from 'hono/router/reg-exp-router'
import { LinearRouter } from 'hono/router/linear-router'

const customApp = new Hono({
  router: new RegExpRouter() // Direct router specification
})

// Or create a custom combination
import { SmartRouter } from 'hono/router/smart-router'

const hybridApp = new Hono({
  router: new SmartRouter({
    routers: [new RegExpRouter(), new LinearRouter()]
  })
})
```

## See Also

- [Router Presets Specification](/docs/api/presets) - Complete formal specification
- [Routing Specification](/docs/api/routing) - Routing pattern details
- [Hono Examples](/docs/api/hono-examples) - Application setup examples