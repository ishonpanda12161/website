# HonoRequest Usage Examples

This page provides practical examples for working with the HonoRequest object in Hono applications.

## Path Parameter Examples

### Single Parameter

```ts
import { Hono } from 'hono'

const app = new Hono()

app.get('/users/:id', async (c) => {
  const id = c.req.param('id')
  return c.json({ userId: id })
})
```

### Multiple Parameters

```ts
app.get('/posts/:id/comments/:commentId', async (c) => {
  const { id, commentId } = c.req.param()
  return c.json({
    postId: id,
    commentId: commentId,
  })
})
```

### Optional Parameters

```ts
app.get('/api/posts/:category?', (c) => {
  const category = c.req.param('category')
  if (category) {
    return c.text(`Posts in category: ${category}`)
  }
  return c.text('All posts')
})
```

## Query String Examples

### Single Query Parameter

```ts
app.get('/search', async (c) => {
  const query = c.req.query('q')
  const limit = c.req.query('limit') || '10'

  return c.json({
    query,
    limit: parseInt(limit),
    results: [],
  })
})
```

### Multiple Query Parameters

```ts
app.get('/filter', async (c) => {
  const { category, sort, page } = c.req.query()

  return c.json({
    filters: { category, sort },
    pagination: { page: parseInt(page || '1') },
  })
})
```

### Array Query Parameters

```ts
app.get('/search', async (c) => {
  // Handle URLs like /search?tags=javascript&tags=web&tags=api
  const tags = c.req.queries('tags')

  return c.json({
    searchTags: tags, // string[]
    resultCount: tags.length,
  })
})
```

## Request Header Examples

### Authentication Headers

```ts
app.get('/protected', (c) => {
  const auth = c.req.header('Authorization')
  const userAgent = c.req.header('User-Agent')

  if (!auth?.startsWith('Bearer ')) {
    return c.text('Unauthorized', 401)
  }

  return c.json({
    message: 'Access granted',
    userAgent,
  })
})
```

### Content Type Detection

```ts
app.post('/upload', async (c) => {
  const contentType = c.req.header('Content-Type')

  if (contentType?.includes('application/json')) {
    const data = await c.req.json()
    return c.json({ type: 'json', data })
  } else if (contentType?.includes('multipart/form-data')) {
    const data = await c.req.parseBody()
    return c.json({ type: 'form', data })
  }

  return c.text('Unsupported content type', 400)
})
```

### Header Case Sensitivity

```ts
app.get('/headers', (c) => {
  // ❌ Won't work - headers() returns lowercase keys
  const headers = c.req.header()
  const customHeader = headers['X-Custom-Header'] // undefined

  // ✅ Correct way - use specific header name
  const customHeader = c.req.header('X-Custom-Header')

  return c.json({ customHeader })
})
```

## Request Body Parsing Examples

### JSON Body

```ts
app.post('/api/users', async (c) => {
  try {
    const body = await c.req.json()

    // Validate required fields
    if (!body.name || !body.email) {
      return c.json({ error: 'Missing required fields' }, 400)
    }

    return c.json(
      {
        message: 'User created',
        user: { id: 123, ...body },
      },
      201
    )
  } catch (error) {
    return c.json({ error: 'Invalid JSON' }, 400)
  }
})
```

### Form Data (Simple)

```ts
app.post('/contact', async (c) => {
  const body = await c.req.parseBody()

  const name = body['name'] as string
  const email = body['email'] as string
  const message = body['message'] as string

  return c.json({
    received: { name, email, message },
  })
})
```

### File Upload (Single File)

```ts
app.post('/upload', async (c) => {
  const body = await c.req.parseBody()

  const file = body['file'] as File
  if (!file) {
    return c.text('No file uploaded', 400)
  }

  return c.json({
    filename: file.name,
    size: file.size,
    type: file.type,
  })
})
```

### Multiple File Upload

```ts
app.post('/gallery', async (c) => {
  const body = await c.req.parseBody()

  // Files with array notation
  const files = body['images[]'] as File[]

  if (!Array.isArray(files)) {
    return c.text('No files uploaded', 400)
  }

  const fileInfo = files.map((file) => ({
    name: file.name,
    size: file.size,
    type: file.type,
  }))

  return c.json({
    uploadedFiles: fileInfo,
    count: files.length,
  })
})
```

### Multiple Fields with Same Name

```ts
app.post('/survey', async (c) => {
  // Handle multiple checkboxes with same name
  const body = await c.req.parseBody({ all: true })

  const interests = body['interests'] // string | string[]
  const normalizedInterests = Array.isArray(interests)
    ? interests
    : [interests].filter(Boolean)

  return c.json({
    userInterests: normalizedInterests,
  })
})
```

### Dot Notation in Form Data

```ts
app.post('/profile', async (c) => {
  const body = await c.req.parseBody({ dot: true })

  // If form data includes: user.name, user.email, address.city, address.zip
  // body will be: { user: { name: '...', email: '...' }, address: { city: '...', zip: '...' } }

  return c.json({
    profile: body,
  })
})
```

## Text and Binary Data Examples

### Plain Text

```ts
app.post('/webhook', async (c) => {
  const payload = await c.req.text()

  // Process webhook payload
  console.log('Webhook received:', payload)

  return c.text('OK')
})
```

### Binary Data (ArrayBuffer)

```ts
app.post('/binary', async (c) => {
  const buffer = await c.req.arrayBuffer()
  const size = buffer.byteLength

  return c.json({
    message: 'Binary data received',
    size,
  })
})
```

### Blob Handling

```ts
app.post('/media', async (c) => {
  const blob = await c.req.blob()

  return c.json({
    type: blob.type,
    size: blob.size,
  })
})
```

### FormData Access

```ts
app.post('/form', async (c) => {
  const formData = await c.req.formData()

  // Direct FormData manipulation
  const username = formData.get('username')
  const files = formData.getAll('attachments')

  return c.json({
    username,
    attachmentCount: files.length,
  })
})
```

## Request Properties Examples

### URL and Path Information

```ts
app.get('/debug/*', (c) => {
  return c.json({
    url: c.req.url, // Full URL
    path: c.req.path, // Path only
    method: c.req.method, // HTTP method
  })
})
```

### Raw Request Access

```ts
// For platform-specific features
app.post('/cf-specific', async (c) => {
  const cfData = c.req.raw.cf // Cloudflare-specific data

  return c.json({
    country: cfData?.country,
    city: cfData?.city,
  })
})
```

## Validation Examples

### Using Validation Middleware

```ts
import { Hono } from 'hono'
import { validator } from 'hono/validator'

const app = new Hono()

app.post(
  '/posts',
  validator('form', (value, c) => {
    if (!value['title'] || !value['body']) {
      return c.text('Title and body are required', 400)
    }
    return {
      title: value['title'],
      body: value['body'],
    }
  }),
  async (c) => {
    const { title, body } = c.req.valid('form')

    return c.json({
      message: 'Post created',
      post: { title, body },
    })
  }
)
```

### JSON Schema Validation

```ts
import { z } from 'zod'
import { zValidator } from '@hono/zod-validator'

const userSchema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
  age: z.number().optional(),
})

app.post('/users', zValidator('json', userSchema), async (c) => {
  const user = c.req.valid('json') // Fully typed!

  return c.json({
    message: 'User created',
    user,
  })
})
```

## Advanced Request Handling

### Conditional Request Processing

```ts
app.all('/api/*', async (c) => {
  const method = c.req.method
  const path = c.req.path

  if (method === 'OPTIONS') {
    return c.text('', 204, {
      'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE',
      'Access-Control-Allow-Headers': 'Content-Type, Authorization',
    })
  }

  // Continue with normal processing
  return c.text(`${method} ${path}`)
})
```

### Request Parsing Strategy

```ts
app.post('/flexible', async (c) => {
  const contentType = c.req.header('Content-Type')

  let data: any

  if (contentType?.includes('application/json')) {
    data = await c.req.json()
  } else if (
    contentType?.includes('application/x-www-form-urlencoded')
  ) {
    data = await c.req.parseBody()
  } else if (contentType?.includes('text/')) {
    data = { text: await c.req.text() }
  } else {
    return c.text('Unsupported content type', 415)
  }

  return c.json({ received: data })
})
```

## See Also

- [HonoRequest API Specification](/docs/api/request) - Complete API reference
- [Context Examples](/docs/api/context-examples) - Context usage patterns
- [Validation Guide](/docs/guides/validation) - Request validation patterns
- [Testing Guide](/docs/guides/testing) - Testing strategies
