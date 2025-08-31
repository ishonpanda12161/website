# HonoRequest Examples

Practical examples and patterns for using HonoRequest methods for parsing and accessing request data.

## Parameter Access Examples

### Path Parameters

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// Single parameter
app.get('/entry/:id', async (c) => {
  const id = c.req.param('id')
  //    ^?
  // ...
})

// Multiple parameters at once
app.get('/entry/:id/comment/:commentId', async (c) => {
  const { id, commentId } = c.req.param()
  //      ^?
})
```

### Query String Parameters

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// Single query parameter
app.get('/search', async (c) => {
  const query = c.req.query('q')
  //     ^?
})

// All query parameters at once
app.get('/search', async (c) => {
  const { q, limit, offset } = c.req.query()
  //      ^?
})
```

### Multiple Query Values

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/search', async (c) => {
  // For URL: /search?tags=A&tags=B
  const tags = c.req.queries('tags')
  //     ^?
  // tags will be ['A', 'B']
})
```

## Header Access Examples

### Basic Header Access

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/', (c) => {
  const userAgent = c.req.header('User-Agent')
  //      ^?
  return c.text(`Your user agent is ${userAgent}`)
})
```

### Header Case Sensitivity

```ts
import { Hono } from 'hono'
const app = new Hono()

app.get('/', (c) => {
  // ❌ Will not work - keys are lowercase when getting all headers
  const headerRecord = c.req.header()
  const foo = headerRecord['X-Foo'] // undefined
  
  // ✅ Will work - direct header access is case-insensitive  
  const foo2 = c.req.header('X-Foo')
  const foo3 = c.req.header('x-foo') // Same result
  
  return c.text('OK')
})
```

## Request Body Parsing Examples

### JSON Body Parsing

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.post('/api/users', async (c) => {
  const body = await c.req.json()
  console.log(body.name) // Access JSON properties
  return c.json({ success: true })
})
```

### Text Body Parsing

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.post('/webhook', async (c) => {
  const body = await c.req.text()
  console.log('Received:', body)
  return c.text('OK')
})
```

### Form Data Parsing

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.post('/contact', async (c) => {
  const body = await c.req.parseBody()
  console.log(body['name']) // Form field access
  return c.redirect('/thank-you')
})
```

## Advanced Form Parsing Examples

### Single File Upload

```ts twoslash
import { Context } from 'hono'
declare const c: Context
// ---cut---
const body = await c.req.parseBody()
const data = body['avatar'] // (string | File)
if (data instanceof File) {
  console.log('File name:', data.name)
  console.log('File size:', data.size)
}
```

### Multiple File Upload

```ts twoslash
import { Context } from 'hono'
declare const c: Context
// ---cut---
const body = await c.req.parseBody()
const files = body['photos[]'] // (string | File)[]
if (Array.isArray(files)) {
  files.forEach((file) => {
    if (file instanceof File) {
      console.log('File:', file.name)
    }
  })
}
```

### Multiple Files with Same Name

```ts twoslash
import { Context } from 'hono'
declare const c: Context
// ---cut---
// For multiple <input type="file" multiple /> or checkboxes
const body = await c.req.parseBody({ all: true })
const favorites = body['favorites'] // (string | File) | (string | File)[]

if (Array.isArray(favorites)) {
  console.log('Multiple values:', favorites)
} else {
  console.log('Single value:', favorites)
}
```

### Dot Notation Parsing

```ts twoslash
import { Context } from 'hono'
declare const c: Context
// ---cut---
// For form data like: obj.key1=value1&obj.key2=value2
const body = await c.req.parseBody({ dot: true })
console.log(body) // { obj: { key1: 'value1', key2: 'value2' } }
```

## Binary Data Examples

### ArrayBuffer Processing

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.post('/upload', async (c) => {
  const buffer = await c.req.arrayBuffer()
  const byteLength = buffer.byteLength
  console.log(`Received ${byteLength} bytes`)
  return c.text('Upload received')
})
```

### Blob Processing

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.post('/image', async (c) => {
  const blob = await c.req.blob()
  console.log('Content type:', blob.type)
  console.log('Size:', blob.size)
  return c.text('Image processed')
})
```

### Direct FormData Access

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.post('/form', async (c) => {
  const formData = await c.req.formData()
  for (const [key, value] of formData.entries()) {
    console.log(key, value)
  }
  return c.text('Form processed')
})
```

## Validation Examples

### Validated Data Access

```ts
import { Hono } from 'hono'
const app = new Hono()

app.post('/posts', async (c) => {
  // Assumes validation middleware is applied
  const { title, body } = c.req.valid('form') as { title: string; body: string }
  const queryParams = c.req.valid('query')
  const headers = c.req.valid('header')
  
  return c.json({ title, body })
})
```

### Different Validation Targets

```ts
import { Hono } from 'hono'
const app = new Hono()

app.post('/api', async (c) => {
  const formData = c.req.valid('form')     // Form validation
  const jsonData = c.req.valid('json')     // JSON validation
  const queryData = c.req.valid('query')   // Query validation
  const headerData = c.req.valid('header') // Header validation
  const cookieData = c.req.valid('cookie') // Cookie validation
  const paramData = c.req.valid('param')   // Path param validation
  
  return c.json({ success: true })
})
```

## Request Metadata Examples

### Basic Request Info

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/debug', async (c) => {
  const path = c.req.path         // '/debug'
  const url = c.req.url           // 'https://example.com/debug'
  const method = c.req.method     // 'GET'
  
  return c.json({ path, url, method })
})
```

### Raw Request Access

```ts
import { Hono } from 'hono'
const app = new Hono()

app.post('/webhook', async (c) => {
  // Access underlying Request object
  const raw = c.req.raw
  
  // Platform-specific features (e.g., Cloudflare Workers)
  const cf = (raw as any).cf // Cloudflare-specific properties
  
  return c.text('OK')
})
```

## Error Handling Examples

### Safe Parameter Access

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/user/:id?', async (c) => {
  const id = c.req.param('id')
  if (!id) {
    return c.text('ID is required', 400)
  }
  
  return c.json({ userId: id })
})
```

### Body Parsing Error Handling

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.post('/api/data', async (c) => {
  try {
    const body = await c.req.json()
    return c.json({ success: true, data: body })
  } catch (error) {
    return c.json({ error: 'Invalid JSON' }, 400)
  }
})
```

### File Upload Validation

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.post('/upload', async (c) => {
  const body = await c.req.parseBody()
  const file = body['file']
  
  if (!(file instanceof File)) {
    return c.json({ error: 'File is required' }, 400)
  }
  
  if (file.size > 1024 * 1024) { // 1MB limit
    return c.json({ error: 'File too large' }, 400)
  }
  
  return c.json({ message: 'File uploaded', filename: file.name })
})
```

## See Also

- [HonoRequest Specification](/docs/api/request) - Complete formal specification
- [Context Examples](/docs/api/context-examples) - Related context usage patterns
- [Validation Guide](/docs/guides/validation) - Request validation patterns