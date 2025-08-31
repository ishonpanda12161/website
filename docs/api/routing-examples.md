# Routing Examples

Practical examples demonstrating the flexible routing capabilities of Hono.

## Basic HTTP Method Routing

### Standard HTTP Methods

```ts
import { Hono } from 'hono'
const app = new Hono()

// Basic HTTP methods
app.get('/', (c) => c.text('GET /'))
app.post('/', (c) => c.text('POST /'))
app.put('/', (c) => c.text('PUT /'))
app.delete('/', (c) => c.text('DELETE /'))
app.patch('/', (c) => c.text('PATCH /'))
app.head('/', (c) => c.text('HEAD /'))
app.options('/', (c) => c.text('OPTIONS /'))
```

### All Methods and Custom Methods

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// Handle all HTTP methods
app.all('/hello', (c) => c.text('Any Method /hello'))

// Custom HTTP method
app.on('PURGE', '/cache', (c) => c.text('PURGE Method /cache'))

// Multiple methods for same route
app.on(['PUT', 'DELETE'], '/post', (c) =>
  c.text('PUT or DELETE /post')
)

// Multiple paths for same handler
app.on('GET', ['/hello', '/ja/hello', '/en/hello'], (c) =>
  c.text('Hello')
)
```

## Path Parameter Patterns

### Basic Parameters

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// Single parameter
app.get('/user/:name', async (c) => {
  const name = c.req.param('name')
  //       ^?
  return c.text(`Hello, ${name}!`)
})

// Multiple parameters
app.get('/posts/:id/comment/:comment_id', async (c) => {
  const { id, comment_id } = c.req.param()
  //       ^?
  return c.json({ postId: id, commentId: comment_id })
})

// Nested parameters
app.get('/users/:userId/posts/:postId/comments/:commentId', async (c) => {
  const params = c.req.param()
  return c.json(params)
})
```

### Optional Parameters

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// Optional parameter - matches both paths
app.get('/api/animal/:type?', (c) => {
  const type = c.req.param('type')
  if (type) {
    return c.text(`Animal type: ${type}`)
  }
  return c.text('All animals')
})

// Multiple optional parameters
app.get('/search/:category?/:subcategory?', (c) => {
  const { category, subcategory } = c.req.param()
  const filters = []
  if (category) filters.push(`category: ${category}`)
  if (subcategory) filters.push(`subcategory: ${subcategory}`)
  
  return c.text(filters.join(', ') || 'All items')
})
```

### Regex Constrained Parameters

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// Numeric IDs only
app.get('/user/:id{[0-9]+}', async (c) => {
  const id = c.req.param('id')
  return c.text(`User ID (numeric): ${id}`)
})

// Date and slug pattern
app.get('/post/:date{[0-9]{4}-[0-9]{2}-[0-9]{2}}/:title{[a-z0-9-]+}', async (c) => {
  const { date, title } = c.req.param()
  //       ^?
  return c.json({ publishDate: date, slug: title })
})

// File extensions
app.get('/static/:filename{.+\\.(css|js|png|jpg)}', async (c) => {
  const filename = c.req.param('filename')
  return c.text(`Serving static file: ${filename}`)
})

// Complex patterns
app.get('/posts/:slug{[a-z0-9-]+}/:action{(edit|delete|view)}', async (c) => {
  const { slug, action } = c.req.param()
  return c.text(`${action} post: ${slug}`)
})
```

## Wildcard Patterns

### Single and Multi-segment Wildcards

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// Single segment wildcard
app.get('/static/*/file', (c) => {
  return c.text('Static file in any subdirectory')
})

// Multi-segment wildcard
app.get('/files/**', (c) => {
  const path = c.req.path
  return c.text(`File at path: ${path}`)
})

// Named wildcard
app.get('/download/*filepath', (c) => {
  const filepath = c.req.param('filepath')
  return c.text(`Download file: ${filepath}`)
})
```

## Method Chaining

### Chained Route Definitions

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// Chain multiple methods for same path
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

// RESTful resource
app
  .get('/posts', (c) => c.json({ message: 'List posts' }))
  .post(async (c) => {
    const body = await c.req.json()
    return c.json({ message: 'Created post', data: body })
  })

app
  .get('/posts/:id', (c) => {
    const id = c.req.param('id')
    return c.json({ message: `Get post ${id}` })
  })
  .put(async (c) => {
    const id = c.req.param('id')
    const body = await c.req.json()
    return c.json({ message: `Updated post ${id}`, data: body })
  })
  .delete((c) => {
    const id = c.req.param('id')
    return c.json({ message: `Deleted post ${id}` })
  })
```

## Route Grouping and Organization

### Sub-Applications

```ts twoslash
import { Hono } from 'hono'
// ---cut---
// Book routes
const books = new Hono()

books.get('/', (c) => c.text('List Books'))
books.get('/:id', (c) => {
  const id = c.req.param('id')
  return c.text('Get Book: ' + id)
})
books.post('/', (c) => c.text('Create Book'))
books.put('/:id', (c) => {
  const id = c.req.param('id')
  return c.text(`Update Book: ${id}`)
})
books.delete('/:id', (c) => {
  const id = c.req.param('id')
  return c.text(`Delete Book: ${id}`)
})

// User routes
const users = new Hono()

users.get('/', (c) => c.text('List Users'))
users.post('/', (c) => c.text('Create User'))
users.get('/:id/profile', (c) => {
  const id = c.req.param('id')
  return c.text(`Profile for user ${id}`)
})

// Main application
const app = new Hono()

app.route('/books', books)   // All book routes available at /books/*
app.route('/users', users)   // All user routes available at /users/*
```

### API Versioning with Grouping

```ts twoslash
import { Hono } from 'hono'
// ---cut---
// API v1
const v1 = new Hono()
v1.get('/users', (c) => c.json({ version: 'v1', users: [] }))
v1.post('/users', (c) => c.json({ version: 'v1', message: 'User created' }))

// API v2
const v2 = new Hono()
v2.get('/users', (c) => c.json({ version: 'v2', users: [], meta: {} }))
v2.post('/users', (c) => c.json({ version: 'v2', message: 'User created with validation' }))

// Mount versioned APIs
const app = new Hono()
app.route('/api/v1', v1)
app.route('/api/v2', v2)

// Default to latest version
app.route('/api', v2)
```

### Base Path Configuration

```ts twoslash
import { Hono } from 'hono'
// ---cut---
// API with base path
const api = new Hono().basePath('/api')
api.get('/users', (c) => c.text('GET /api/users'))
api.get('/posts', (c) => c.text('GET /api/posts'))

// Admin routes with base path
const admin = new Hono().basePath('/admin')
admin.get('/dashboard', (c) => c.text('Admin Dashboard'))
admin.get('/users', (c) => c.text('Admin Users'))

// Combine multiple base paths
const app = new Hono()
app.route('/', api)    // Mounts /api/* routes
app.route('/', admin)  // Mounts /admin/* routes
```

### Multiple Grouping Styles

```ts twoslash
import { Hono } from 'hono'
// ---cut---
// Style 1: Each group defines its own full paths
const blogRoutes = new Hono()
blogRoutes.get('/blog', (c) => c.text('Blog Home'))
blogRoutes.get('/blog/:slug', (c) => {
  const slug = c.req.param('slug')
  return c.text(`Blog Post: ${slug}`)
})

// Style 2: Group uses relative paths with basePath
const shopRoutes = new Hono().basePath('/shop')
shopRoutes.get('/', (c) => c.text('Shop Home'))
shopRoutes.get('/products', (c) => c.text('Products'))
shopRoutes.get('/cart', (c) => c.text('Shopping Cart'))

// Main app mounts both styles
const app = new Hono()
app.route('/', blogRoutes)  // Handles /blog/*
app.route('/', shopRoutes)  // Handles /shop/*
```

## Advanced Routing Scenarios

### Hostname-Based Routing

```ts twoslash
import { Hono } from 'hono'
// ---cut---
const app = new Hono({
  getPath: (req) => req.url.replace(/^https?:\/([^?]+).*$/, '$1'),
})

app.get('/api.example.com/users', (c) => c.json({ api: 'users' }))
app.get('/admin.example.com/dashboard', (c) => c.text('Admin Dashboard'))
app.get('/blog.example.com/posts', (c) => c.json({ posts: [] }))

// Catch-all for main domain
app.get('/www.example.com/*', (c) => c.text('Main website'))
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

app.get('/api.example.com/health', (c) => c.json({ status: 'ok' }))
app.get('/cdn.example.com/assets/*', (c) => c.text('CDN Asset'))

// This will match requests with matching host headers
// new Request('http://api.example.com/health', {
//   headers: { host: 'api.example.com' }
// })
```

### User-Agent Based Routing

```ts twoslash
import { Hono } from 'hono'
// ---cut---
const app = new Hono({
  getPath: (req) => {
    const userAgent = req.headers.get('user-agent') || ''
    const isMobile = /Mobile|Android|iPhone/i.test(userAgent)
    const prefix = isMobile ? '/mobile' : '/desktop'
    const path = new URL(req.url).pathname
    return prefix + path
  }
})

app.get('/mobile/home', (c) => c.html('<h1>Mobile Home</h1>'))
app.get('/desktop/home', (c) => c.html('<h1>Desktop Home</h1>'))
```

## Route Priority and Execution Order

### Priority Examples

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// Specific routes first
app.get('/users/me', (c) => c.text('Current user'))
app.get('/users/admin', (c) => c.text('Admin user'))
app.get('/users/:id', (c) => {
  const id = c.req.param('id')
  return c.text(`User ${id}`)
})

// Results:
// GET /users/me    -> "Current user"
// GET /users/admin -> "Admin user"  
// GET /users/123   -> "User 123"
```

### Middleware and Handler Ordering

```ts twoslash
import { Hono } from 'hono'
import { logger } from 'hono/logger'
import { cors } from 'hono/cors'
const app = new Hono()
// ---cut---
// Global middleware first
app.use(logger())
app.use(cors())

// Path-specific middleware
app.use('/api/*', async (c, next) => {
  console.log('API middleware')
  await next()
})

// Routes after middleware
app.get('/api/users', (c) => c.json({ users: [] }))

// Fallback routes last
app.get('*', (c) => c.text('Fallback handler'))
```

### Early Route Termination

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// This will shadow later routes
app.get('*', (c) => c.text('Catch all')) // Executes first!

app.get('/specific', (c) => c.text('Specific route')) // Never reached!

// Correct order:
app.get('/specific', (c) => c.text('Specific route'))
app.get('*', (c) => c.text('Catch all'))
```

## Common Route Patterns

### RESTful API Routes

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// RESTful posts API
app.get('/posts', (c) => c.json({ posts: [] }))
app.post('/posts', async (c) => {
  const post = await c.req.json()
  return c.json({ created: post }, 201)
})
app.get('/posts/:id', (c) => {
  const id = c.req.param('id')
  return c.json({ post: { id } })
})
app.put('/posts/:id', async (c) => {
  const id = c.req.param('id')
  const updates = await c.req.json()
  return c.json({ updated: { id, ...updates } })
})
app.delete('/posts/:id', (c) => {
  const id = c.req.param('id')
  return c.json({ deleted: id })
})

// Nested resources
app.get('/posts/:postId/comments', (c) => {
  const postId = c.req.param('postId')
  return c.json({ comments: [], postId })
})
app.post('/posts/:postId/comments', async (c) => {
  const postId = c.req.param('postId')
  const comment = await c.req.json()
  return c.json({ created: comment, postId }, 201)
})
```

### File and Asset Serving

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// Static file patterns
app.get('/static/:filename{.+\\.(css|js)}', (c) => {
  const filename = c.req.param('filename')
  // Serve CSS/JS files
  return c.text(`/* Serving ${filename} */`)
})

app.get('/images/:filename{.+\\.(jpg|png|gif|webp)}', (c) => {
  const filename = c.req.param('filename')
  // Serve image files
  return c.text(`Image: ${filename}`)
})

app.get('/downloads/:filename{.+}', (c) => {
  const filename = c.req.param('filename')
  return c.text(`Download: ${filename}`)
})
```

### Authentication Route Patterns

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// Auth routes
app.post('/auth/login', (c) => c.json({ message: 'Login' }))
app.post('/auth/logout', (c) => c.json({ message: 'Logout' }))
app.post('/auth/register', (c) => c.json({ message: 'Register' }))
app.post('/auth/forgot-password', (c) => c.json({ message: 'Reset sent' }))
app.post('/auth/reset-password/:token{[a-f0-9]{32}}', (c) => {
  const token = c.req.param('token')
  return c.json({ message: 'Password reset', token })
})

// Protected routes with auth middleware
app.use('/protected/*', async (c, next) => {
  // Auth middleware logic
  await next()
})

app.get('/protected/profile', (c) => c.json({ profile: 'user data' }))
app.get('/protected/settings', (c) => c.json({ settings: {} }))
```

## Grouping Order Pitfalls

### Correct Grouping Order

```ts twoslash
import { Hono } from 'hono'
// ---cut---
const nested = new Hono()
const middle = new Hono()
const app = new Hono()

// Configure routes first
nested.get('/endpoint', (c) => c.text('Nested endpoint'))

// Then establish hierarchy
middle.route('/nested', nested)
app.route('/middle', middle)

// Result: GET /middle/nested/endpoint -> "Nested endpoint"
```

### Incorrect Order (404 Error)

```ts twoslash
import { Hono } from 'hono'
// ---cut---
const nested = new Hono()
const middle = new Hono()
const app = new Hono()

// Wrong: mount before configuring
app.route('/middle', middle)  // middle has no routes yet!
middle.route('/nested', nested)
nested.get('/endpoint', (c) => c.text('Nested endpoint'))

// Result: GET /middle/nested/endpoint -> 404 Not Found
```

## See Also

- [Routing Specification](/docs/api/routing) - Complete routing system reference
- [Hono Examples](/docs/api/hono-examples) - Application setup and organization
- [Context Examples](/docs/api/context-examples) - Handler context usage
- [Request Examples](/docs/api/request-examples) - Parameter access patterns