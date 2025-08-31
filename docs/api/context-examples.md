# Context Examples

This page collects practical patterns and snippets that complement the formal [Context Specification](/docs/api/context).

## Mutating headers on the final Response

Append headers after the handler finishes using middleware and `await next()`:

```ts
app.use('/', async (c, next) => {
  await next()
  c.res.headers.append('X-Debug', 'Debug message')
})
```

## Variables via createMiddleware and Env generics

```ts
import { Hono } from 'hono'
import { createMiddleware } from 'hono/factory'

type Env = { Variables: { echo: (s: string) => string } }

const echoMiddleware = createMiddleware<Env>(async (c, next) => {
  c.set('echo', (str) => str)
  await next()
})

const app = new Hono<Env>()

app.use(echoMiddleware)

app.get('/', (c) => {
  return c.text(c.var.echo('Hello!'))
})
```

## JSX rendering with setRenderer and render

```ts
import { Hono } from 'hono'
import { html } from 'hono/html'

const app = new Hono()

app.use('*', async (c, next) => {
  c.setRenderer((content, head) => {
    return c.html(html`
      <!DOCTYPE html>
      <html>
        <head>
          ${head?.title ? html`<title>${head.title}</title>` : ''}
        </head>
        <body>
          ${content}
        </body>
      </html>
    `)
  })
  await next()
})

app.get('/', (c) => {
  return c.render(html`<h1>Hello JSX!</h1>`, {
    title: 'Top page'
  })
})
```

## Cloudflare executionCtx and event usage

```ts
// executionCtx for controlling request lifecycle
app.get('/background-task', async (c) => {
  // Don't wait for this to complete before responding
  c.executionCtx.waitUntil(performBackgroundWork())
  
  return c.text('Started background work')
})

// event object usage (deprecated, use executionCtx instead)
app.get('/legacy', async (c) => {
  if (c.event) {
    c.event.waitUntil(legacyBackgroundTask())
  }
  return c.text('Legacy pattern')
})
```

## ContextVariableMap augmentation

Extend the type system for middleware-provided variables:

```ts
declare module 'hono' {
  interface ContextVariableMap {
    user: {
      id: string
      name: string
    }
    requestId: string
    logger: {
      info: (msg: string) => void
      error: (msg: string) => void
    }
  }
}

// Now c.var.user, c.var.requestId, and c.var.logger are properly typed
app.use('*', async (c, next) => {
  c.set('requestId', crypto.randomUUID())
  c.set('logger', {
    info: (msg) => console.log(`[${c.get('requestId')}] ${msg}`),
    error: (msg) => console.error(`[${c.get('requestId')}] ${msg}`)
  })
  await next()
})

app.get('/', (c) => {
  c.var.logger.info('Processing request')
  return c.text(`Request ID: ${c.var.requestId}`)
})
```

## Working with request body parsing

```ts
// Multiple parsing approaches
app.post('/form', async (c) => {
  const body = await c.req.parseBody()
  // body is Record<string, string | File>
  return c.json({ received: Object.keys(body) })
})

app.post('/json', async (c) => {
  const data = await c.req.json()
  // data is any, consider validation
  return c.json({ echo: data })
})

app.post('/text', async (c) => {
  const text = await c.req.text()
  return c.text(`Received: ${text.length} characters`)
})
```

## Error handling patterns

```ts
// Throwing HTTP exceptions
app.get('/protected', (c) => {
  const auth = c.req.header('authorization')
  if (!auth) {
    throw new HTTPException(401, { message: 'Unauthorized' })
  }
  return c.text('Protected content')
})

// Accessing error in middleware
app.onError((err, c) => {
  if (err instanceof HTTPException) {
    return err.getResponse()
  }
  
  console.error('Unexpected error:', err)
  return c.text('Internal Server Error', 500)
})
```

## Platform-specific patterns

```ts
// Cloudflare Workers specific
app.get('/kv', async (c) => {
  const value = await c.env.MY_KV.get('key')
  return c.json({ value })
})

// Access Cloudflare request properties
app.get('/cf-info', (c) => {
  const cf = c.req.raw.cf
  return c.json({
    country: cf?.country,
    city: cf?.city,
    timezone: cf?.timezone
  })
})
```