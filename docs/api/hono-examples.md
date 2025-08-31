# Hono Application Examples

This page provides practical examples for building applications with Hono.

## Basic Application Setup

### Simple Web Server

```ts
import { Hono } from 'hono'

const app = new Hono()

app.get('/', (c) => c.text('Hello Hono!'))
app.post('/users', (c) => c.json({ message: 'User created' }))

export default app // for Cloudflare Workers or Bun
```

### Platform-Specific Entry Points

#### Cloudflare Workers

```ts
import { Hono } from 'hono'

const app = new Hono()

app.get('/', (c) => c.text('Hello World'))

export default {
  fetch(request: Request, env: Env, ctx: ExecutionContext) {
    return app.fetch(request, env, ctx)
  },
}
```

Or simply:

```ts
export default app
```

#### Bun

```ts
import { Hono } from 'hono'

const app = new Hono()

app.get('/', (c) => c.text('Hello Bun!'))

export default {
  port: 3000,
  fetch: app.fetch,
}
```

## Error Handling Examples

### Custom 404 Handler

```ts
import { Hono } from 'hono'

const app = new Hono()

app.notFound((c) => {
  return c.text('Custom 404 Message', 404)
})
```

### Global Error Handler

```ts
import { Hono } from 'hono'
import { HTTPException } from 'hono/http-exception'

const app = new Hono()

app.onError((err, c) => {
  if (err instanceof HTTPException) {
    return err.getResponse()
  }
  console.error(`${err}`)
  return c.text('Custom Error Message', 500)
})
```

## Testing Examples

### Unit Testing with request()

```ts
import { describe, expect, test } from 'vitest'
import { Hono } from 'hono'

describe('Example', () => {
  test('GET /', async () => {
    const app = new Hono()
    app.get('/', (c) => c.text('Please test me'))

    const res = await app.request('http://localhost/')
    expect(res.status).toBe(200)
    expect(await res.text()).toBe('Please test me')
  })

  test('POST with body', async () => {
    const app = new Hono()
    app.post('/posts', async (c) => {
      const body = await c.req.json()
      return c.json({ id: body.id }, 201)
    })

    const req = new Request('http://localhost/posts', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ id: 123, title: 'Hello' }),
    })

    const res = await app.request(req)
    expect(res.status).toBe(201)
  })
})
```

## Generic Type Examples

### Environment Bindings and Variables

```ts
import { Hono } from 'hono'

type Bindings = {
  TOKEN: string
  DB: any // KV namespace
}

type Variables = {
  user: User
  session: Session
}

const app = new Hono<{
  Bindings: Bindings
  Variables: Variables
}>()

app.use('/auth/*', async (c, next) => {
  const token = c.env.TOKEN // token is `string`
  const user = await authenticateUser(token)
  c.set('user', user) // user should be `User`
  await next()
})

app.get('/profile', (c) => {
  const user = c.get('user') // Type-safe access to user
  return c.json(user)
})
```

## Router Configuration Examples

### Custom Router Selection

```ts
import { Hono } from 'hono'
import { RegExpRouter } from 'hono/router/reg-exp-router'

const app = new Hono({
  router: new RegExpRouter(),
})
```

### Application Mounting

```ts
import { Hono } from 'hono'

const api = new Hono()
api.get('/users', (c) => c.json([]))
api.get('/posts', (c) => c.json([]))

const app = new Hono()
app.mount('/api', api)

// Now /api/users and /api/posts are available
```

## Advanced Patterns

### Middleware Composition

```ts
import { Hono } from 'hono'
import { cors } from 'hono/cors'
import { logger } from 'hono/logger'
import { prettyJSON } from 'hono/pretty-json'

const app = new Hono()

// Global middleware
app.use('*', logger())
app.use('*', cors())

// API-specific middleware
app.use('/api/*', prettyJSON())

app.get('/api/users', (c) => {
  return c.json([
    { id: 1, name: 'Alice' },
    { id: 2, name: 'Bob' },
  ])
})
```

### Conditional Middleware

```ts
import { Hono } from 'hono'

const app = new Hono()

// Conditional authentication
app.use('/admin/*', async (c, next) => {
  const auth = c.req.header('Authorization')
  if (!auth) {
    return c.text('Unauthorized', 401)
  }
  await next()
})

app.get('/admin/dashboard', (c) => {
  return c.text('Admin Dashboard')
})
```

## See Also

- [API Reference](/docs/api/) - Complete API specification
- [Context Examples](/docs/api/context-examples) - Context usage patterns
- [Request Examples](/docs/api/request-examples) - Request handling examples
- [Routing Examples](/docs/api/routing-examples) - Advanced routing patterns
