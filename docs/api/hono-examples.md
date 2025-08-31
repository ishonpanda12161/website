# Hono Application Examples

Practical usage examples and patterns for the Hono application class.

## Basic Setup

### Simple Application

```ts twoslash
import { Hono } from 'hono'

const app = new Hono()

app.get('/', (c) => c.text('Hello Hono!'))

export default app // for Cloudflare Workers or Bun
```

### With TypeScript Generics

```ts twoslash
import { Hono } from 'hono'
type User = any
declare const user: User
// ---cut---
type Bindings = {
  TOKEN: string
  DB: D1Database
}

type Variables = {
  user: User
  requestId: string
}

const app = new Hono<{
  Bindings: Bindings
  Variables: Variables
}>()

app.use('/auth/*', async (c, next) => {
  const token = c.env.TOKEN // token is `string`
  const db = c.env.DB // db is `D1Database`
  // ...
  c.set('user', user) // user should be `User`
  c.set('requestId', crypto.randomUUID())
  await next()
})
```

## Runtime Platform Integration

### Cloudflare Workers

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
type Env = any
type ExecutionContext = any
// ---cut---
export default {
  fetch(request: Request, env: Env, ctx: ExecutionContext) {
    return app.fetch(request, env, ctx)
  },
}
```

Or simply:

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
export default app
```

### Bun

<!-- prettier-ignore -->
```ts
export default app // [!code --]
export default {  // [!code ++]
  port: 3000, // [!code ++]
  fetch: app.fetch, // [!code ++]
} // [!code ++]
```

### Node.js

```ts
import { serve } from '@hono/node-server'
import { Hono } from 'hono'

const app = new Hono()

app.get('/', (c) => c.text('Hello Node.js!'))

serve(app, (info) => {
  console.log(`Listening on http://localhost:${info.port}`)
})
```

### Deno

```ts
import { Hono } from 'hono'

const app = new Hono()

app.get('/', (c) => c.text('Hello Deno!'))

Deno.serve(app.fetch)
```

## Error Handling

### Custom 404 Handler

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.notFound((c) => {
  return c.text('Custom 404 Message', 404)
})
```

### Global Error Handler

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.onError((err, c) => {
  console.error(`${err}`)
  return c.text('Custom Error Message', 500)
})
```

### HTTPException Handling

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
  console.error(err)
  return c.text('Internal Server Error', 500)
})
```

## Configuration Options

### Strict Mode

```ts twoslash
import { Hono } from 'hono'
// ---cut---
// Strict mode disabled - /hello and /hello/ are the same
const app = new Hono({ strict: false })

app.get('/hello', (c) => c.text('Hello'))
// Matches both /hello and /hello/
```

### Custom Router

```ts twoslash
import { Hono } from 'hono'
// ---cut---
import { RegExpRouter } from 'hono/router/reg-exp-router'

const app = new Hono({ router: new RegExpRouter() })
```

### Custom Path Extraction

```ts twoslash
import { Hono } from 'hono'
// ---cut---
// Hostname-based routing
const app = new Hono({
  getPath: (req) => req.url.replace(/^https?:\/([^?]+).*$/, '$1'),
})

app.get('/www1.example.com/hello', (c) => c.text('hello www1'))
app.get('/www2.example.com/hello', (c) => c.text('hello www2'))
```

### Host Header Routing

```ts twoslash
import { Hono } from 'hono'
// ---cut---
const app = new Hono({
  getPath: (req) =>
    '/' +
    req.headers.get('host') +
    req.url.replace(/^https?:\/\/[^/]+(\/[^?]*).*/, '$1'),
})

app.get('/www1.example.com/hello', (c) => c.text('hello www1'))

// Matches requests like:
// new Request('http://www1.example.com/hello', {
//   headers: { host: 'www1.example.com' },
// })
```

## Application Organization

### Sub-Applications

```ts twoslash
import { Hono } from 'hono'
// ---cut---
const book = new Hono()

book.get('/', (c) => c.text('List Books')) // GET /book
book.get('/:id', (c) => {
  // GET /book/:id
  const id = c.req.param('id')
  return c.text('Get Book: ' + id)
})
book.post('/', (c) => c.text('Create Book')) // POST /book

const app = new Hono()
app.route('/book', book)
```

### Base Path

```ts twoslash
import { Hono } from 'hono'
// ---cut---
const api = new Hono().basePath('/api')
api.get('/book', (c) => c.text('List Books')) // GET /api/book
```

### Multiple Sub-Applications

```ts twoslash
import { Hono } from 'hono'
// ---cut---
const book = new Hono()
book.get('/book', (c) => c.text('List Books')) // GET /book
book.post('/book', (c) => c.text('Create Book')) // POST /book

const user = new Hono().basePath('/user')
user.get('/', (c) => c.text('List Users')) // GET /user
user.post('/', (c) => c.text('Create User')) // POST /user

const app = new Hono()
app.route('/', book) // Handle /book
app.route('/', user) // Handle /user
```

### Mounting External Applications

```ts
import { Router as IttyRouter } from 'itty-router'
import { Hono } from 'hono'

// Create itty-router application
const ittyRouter = IttyRouter()

// Handle `GET /itty-router/hello`
ittyRouter.get('/hello', () => new Response('Hello from itty-router'))

// Hono application
const app = new Hono()

// Mount!
app.mount('/itty-router', ittyRouter.handle)
```

## Testing

### Basic Testing

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
declare const test: (name: string, fn: () => void) => void
declare const expect: (value: any) => any
// ---cut---
test('GET /hello is ok', async () => {
  const res = await app.request('/hello')
  expect(res.status).toBe(200)
})
```

### Testing with Request Object

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
declare const test: (name: string, fn: () => void) => void
declare const expect: (value: any) => any
// ---cut---
test('POST /message is ok', async () => {
  const req = new Request('http://localhost/message', {
    method: 'POST',
    body: 'Hello!',
  })
  const res = await app.request(req)
  expect(res.status).toBe(201)
})
```

### Testing with Environment

```ts twoslash
import { Hono } from 'hono'
declare const test: (name: string, fn: () => void) => void
declare const expect: (value: any) => any

type Bindings = {
  TOKEN: string
}

const app = new Hono<{ Bindings: Bindings }>()

app.get('/secure', (c) => {
  const token = c.env.TOKEN
  return c.text(`Token: ${token}`)
})
// ---cut---
test('Environment bindings work', async () => {
  const res = await app.request('/secure', {}, {
    TOKEN: 'test-token'
  })
  const text = await res.text()
  expect(text).toBe('Token: test-token')
})
```

## Advanced Patterns

### Middleware Composition

```ts twoslash
import { Hono } from 'hono'
import { logger } from 'hono/logger'
import { cors } from 'hono/cors'
// ---cut---
const app = new Hono()

// Global middleware
app.use(logger())
app.use(cors())

// Route-specific middleware
app.use('/api/*', async (c, next) => {
  // API-specific middleware
  await next()
})

app.get('/api/users', (c) => c.json({ users: [] }))
```

### Chained Routes

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app
  .get('/endpoint', (c) => {
    return c.text('GET /endpoint')
  })
  .post((c) => {
    return c.text('POST /endpoint')
  })
  .delete((c) => {
    return c.text('DELETE /endpoint')
  })
```

### Service Worker Integration (Legacy)

::: warning
**Deprecated:** Use `fire()` from `hono/service-worker` instead.
:::

```ts
import { Hono } from 'hono'

const app = new Hono()

app.get('/', (c) => c.text('Hello Service Worker!'))

// Deprecated - use hono/service-worker instead
app.fire()
```

## See Also

- [Hono Specification](/docs/api/hono) - Complete API reference
- [Context Examples](/docs/api/context-examples) - Request/response handling
- [Routing Examples](/docs/api/routing-examples) - Route patterns and organization
- [Middleware Guide](/docs/guides/middleware) - Using and creating middleware