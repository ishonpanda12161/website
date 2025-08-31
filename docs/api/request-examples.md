# HonoRequest Examples

Practical usage examples for the HonoRequest class methods and properties.

## Path Parameters

### Basic Parameter Access

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// Single parameter
app.get('/entry/:id', async (c) => {
  const id = c.req.param('id')
  //    ^?
  return c.text(`Entry ID: ${id}`)
})

// Multiple parameters
app.get('/entry/:id/comment/:commentId', async (c) => {
  const { id, commentId } = c.req.param()
  //      ^?
  return c.text(`Entry ${id}, Comment ${commentId}`)
})
```

### Optional Parameters

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/api/animal/:type?', (c) => {
  const type = c.req.param('type')
  if (type) {
    return c.text(`Animal type: ${type}`)
  }
  return c.text('All animals')
})
```

### Regex Parameters

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/post/:date{[0-9]+}/:title{[a-z]+}', async (c) => {
  const { date, title } = c.req.param()
  //       ^?
  return c.text(`Post from ${date}: ${title}`)
})
```

## Query Parameters

### Basic Query Access

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// Single query parameter
app.get('/search', async (c) => {
  const query = c.req.query('q')
  //     ^?
  return c.text(`Searching for: ${query || 'nothing'}`)
})

// Multiple query parameters
app.get('/search', async (c) => {
  const { q, limit, offset } = c.req.query()
  //      ^?
  return c.json({ query: q, limit, offset })
})
```

### Multiple Values for Same Key

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// URL: /search?tags=A&tags=B&tags=C
app.get('/search', async (c) => {
  // Get only first value
  const firstTag = c.req.query('tags') // "A"
  
  // Get all values
  const allTags = c.req.queries('tags') // ["A", "B", "C"]
  //    ^?
  
  return c.json({ firstTag, allTags })
})

// Get all query parameters with all values
app.get('/search', async (c) => {
  const allParams = c.req.queries()
  // If URL is /search?tags=A&tags=B&name=John
  // allParams = { tags: ["A", "B"], name: ["John"] }
  return c.json(allParams)
})
```

## Headers

### Basic Header Access

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/', (c) => {
  const userAgent = c.req.header('User-Agent')
  //      ^?
  const authorization = c.req.header('Authorization')
  
  return c.text(`Your user agent is ${userAgent}`)
})
```

### All Headers (Lowercase Keys)

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/debug', (c) => {
  const headers = c.req.header()
  
  // ❌ Won't work - keys are lowercase
  const contentType = headers['Content-Type'] // undefined
  
  // ✅ Works - use lowercase key
  const contentType2 = headers['content-type']
  
  // ✅ Better - use direct access
  const contentType3 = c.req.header('Content-Type')
  
  return c.json({ headers })
})
```

## Body Parsing

### JSON Body

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.post('/api/users', async (c) => {
  try {
    const body = await c.req.json()
    // body is typed as any, cast to your type
    const user = body as { name: string; email: string }
    
    return c.json({ message: 'User created', user })
  } catch (e) {
    return c.json({ error: 'Invalid JSON' }, 400)
  }
})
```

### Text Body

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.post('/webhook', async (c) => {
  const body = await c.req.text()
  console.log('Webhook received:', body)
  return c.text('OK')
})
```

### Form Data

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// Simple form
app.post('/contact', async (c) => {
  const body = await c.req.parseBody()
  const name = body['name'] as string
  const email = body['email'] as string
  
  return c.text(`Thanks ${name}! We'll contact you at ${email}`)
})
```

### File Uploads

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// Single file upload
app.post('/upload', async (c) => {
  const body = await c.req.parseBody()
  const file = body['file'] as File
  
  if (file) {
    console.log(`Got file ${file.name} (${file.size} bytes)`)
    return c.text(`Uploaded ${file.name}`)
  }
  
  return c.text('No file uploaded', 400)
})

// Multiple files with same name
app.post('/upload-multiple', async (c) => {
  const body = await c.req.parseBody()
  const files = body['files[]'] as File[] // Note the [] suffix
  
  if (files && Array.isArray(files)) {
    const fileNames = files.map(f => f.name)
    return c.json({ uploaded: fileNames })
  }
  
  return c.text('No files uploaded', 400)
})
```

### Advanced Form Parsing

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// Parse all values (including duplicates)
app.post('/form-all', async (c) => {
  const body = await c.req.parseBody({ all: true })
  
  // If form has multiple checkboxes with same name
  // <input type="checkbox" name="interests" value="sports">
  // <input type="checkbox" name="interests" value="music">
  
  const interests = body['interests']
  // interests will be string[] if multiple selected
  // interests will be string if single selected
  
  return c.json({ interests })
})

// Dot notation parsing
app.post('/nested', async (c) => {
  const body = await c.req.parseBody({ dot: true })
  
  // If form has:
  // <input name="user.name" value="John">
  // <input name="user.email" value="john@example.com">
  
  // body will be: { user: { name: "John", email: "john@example.com" } }
  return c.json(body)
})
```

### Binary Data

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// Handle binary data
app.post('/binary', async (c) => {
  const buffer = await c.req.arrayBuffer()
  const blob = await c.req.blob()
  
  console.log(`Received ${buffer.byteLength} bytes`)
  console.log(`Blob type: ${blob.type}`)
  
  return c.text('Binary data processed')
})

// Native FormData
app.post('/formdata', async (c) => {
  const formData = await c.req.formData()
  
  for (const [key, value] of formData.entries()) {
    if (value instanceof File) {
      console.log(`File ${key}: ${value.name}`)
    } else {
      console.log(`Field ${key}: ${value}`)
    }
  }
  
  return c.text('Form processed')
})
```

## Validation Integration

### Using with Validation Middleware

```ts twoslash
import { Hono } from 'hono'
import { validator } from 'hono/validator'
const app = new Hono()
// ---cut---
// With validation middleware
app.post('/posts', 
  validator('json', (value, c) => {
    const parsed = value as { title?: string; content?: string }
    if (!parsed.title || !parsed.content) {
      return c.text('Missing required fields', 400)
    }
    return { title: parsed.title, content: parsed.content }
  }),
  async (c) => {
    const { title, content } = c.req.valid('json')
    // title and content are guaranteed to exist and be strings
    
    return c.json({ 
      message: 'Post created',
      post: { title, content }
    })
  }
)
```

### Multiple Validation Targets

```ts twoslash
import { Hono } from 'hono'
import { validator } from 'hono/validator'
const app = new Hono()
// ---cut---
app.post('/users/:id/posts',
  validator('param', (value, c) => {
    const parsed = value as { id?: string }
    const id = parseInt(parsed.id || '0')
    if (isNaN(id) || id <= 0) {
      return c.text('Invalid user ID', 400)
    }
    return { id }
  }),
  validator('json', (value, c) => {
    // Validate JSON body
    return value as { title: string }
  }),
  validator('query', (value, c) => {
    // Validate query params
    return value as { draft?: string }
  }),
  async (c) => {
    const { id } = c.req.valid('param')
    const { title } = c.req.valid('json')
    const { draft } = c.req.valid('query')
    
    const isDraft = draft === 'true'
    
    return c.json({ userId: id, title, isDraft })
  }
)
```

## Request Properties

### URL Information

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/debug', (c) => {
  const info = {
    method: c.req.method,        // "GET"
    path: c.req.path,           // "/debug"
    url: c.req.url,             // "https://example.com/debug"
  }
  
  return c.json(info)
})
```

### Route Information (Deprecated)

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/posts/:id', (c) => {
  // ⚠️ Deprecated - use Route Helper instead
  const routePath = c.req.routePath  // "/posts/:id"
  const matched = c.req.matchedRoutes
  
  return c.json({ 
    path: c.req.path,        // "/posts/123"
    routePath,               // "/posts/:id"
    matched
  })
})
```

## Platform-Specific Features

### Cloudflare Workers

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/cf-info', (c) => {
  // Access Cloudflare-specific request properties
  const cf = c.req.raw.cf
  
  if (cf) {
    return c.json({
      country: cf.country,
      city: cf.city,
      region: cf.region,
      timezone: cf.timezone,
      colo: cf.colo
    })
  }
  
  return c.text('Not running on Cloudflare')
})
```

### Raw Request Access

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.post('/raw-request', async (c) => {
  const raw = c.req.raw
  
  // Access native Request methods
  const clone = raw.clone()
  const headers = raw.headers
  
  // Get request body as ReadableStream
  const stream = raw.body
  
  return c.text('Accessed raw request')
})
```

## Error Handling

### JSON Parsing Errors

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.post('/api/data', async (c) => {
  try {
    const data = await c.req.json()
    return c.json({ received: data })
  } catch (error) {
    if (error instanceof SyntaxError) {
      return c.json({ error: 'Invalid JSON format' }, 400)
    }
    return c.json({ error: 'Failed to parse request' }, 500)
  }
})
```

### Form Parsing Errors

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.post('/upload', async (c) => {
  try {
    const body = await c.req.parseBody()
    return c.json({ success: true, fields: Object.keys(body) })
  } catch (error) {
    return c.json({ 
      error: 'Failed to parse form data',
      message: error instanceof Error ? error.message : 'Unknown error'
    }, 400)
  }
})
```

### Missing Data Handling

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/user/:id', (c) => {
  const id = c.req.param('id')
  const format = c.req.query('format')
  
  if (!id) {
    return c.json({ error: 'User ID is required' }, 400)
  }
  
  const response = { id }
  
  // format is optional
  if (format === 'detailed') {
    return c.json({ ...response, details: true })
  }
  
  return c.json(response)
})
```

## Testing Examples

### Unit Testing Request Parsing

```ts twoslash
import { Hono } from 'hono'
declare const test: (name: string, fn: () => void) => void
declare const expect: (value: any) => any

const app = new Hono()

app.post('/test', async (c) => {
  const data = await c.req.json()
  return c.json({ received: data })
})
// ---cut---
test('JSON parsing works', async () => {
  const payload = { name: 'test', value: 123 }
  const req = new Request('http://localhost/test', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(payload)
  })
  
  const res = await app.request(req)
  const result = await res.json()
  
  expect(result.received).toEqual(payload)
})
```

### Testing Form Data

```ts twoslash
import { Hono } from 'hono'
declare const test: (name: string, fn: () => void) => void
declare const expect: (value: any) => any

const app = new Hono()

app.post('/form', async (c) => {
  const body = await c.req.parseBody()
  return c.json(body)
})
// ---cut---
test('Form parsing works', async () => {
  const formData = new FormData()
  formData.append('name', 'John')
  formData.append('age', '30')
  
  const req = new Request('http://localhost/form', {
    method: 'POST',
    body: formData
  })
  
  const res = await app.request(req)
  const result = await res.json()
  
  expect(result.name).toBe('John')
  expect(result.age).toBe('30')
})
```

## See Also

- [HonoRequest Specification](/docs/api/request) - Complete API reference
- [Context Examples](/docs/api/context-examples) - Context object usage
- [Validation Guide](/docs/guides/validation) - Input validation patterns
- [Routing Examples](/docs/api/routing-examples) - Path parameter patterns