# Routing Usage Examples

This page provides practical examples for implementing various routing patterns in Hono applications.

## Basic Routing Examples

### HTTP Method Routing

```ts
import { Hono } from 'hono'

const app = new Hono()

// Standard HTTP methods
app.get('/', (c) => c.text('GET /'))
app.post('/', (c) => c.text('POST /'))
app.put('/', (c) => c.text('PUT /'))
app.delete('/', (c) => c.text('DELETE /'))

// Handle any HTTP method
app.all('/ping', (c) => c.text(`${c.req.method} /ping`))

// Custom HTTP methods
app.on('PURGE', '/cache', (c) => c.text('Cache purged'))

// Multiple methods for same route
app.on(['PUT', 'PATCH'], '/users/:id', (c) =>
  c.text(`${c.req.method} /users/${c.req.param('id')}`)
)

// Multiple paths for same handler
app.on('GET', ['/hello', '/hi', '/greetings'], (c) =>
  c.text('Hello there!')
)
```

## Path Parameter Examples

### Basic Parameters

```ts
app.get('/users/:id', async (c) => {
  const id = c.req.param('id')
  return c.json({
    user: { id, name: `User ${id}` },
  })
})
```

### Multiple Parameters

```ts
app.get('/posts/:id/comments/:commentId', async (c) => {
  const { id, commentId } = c.req.param()

  return c.json({
    post: id,
    comment: commentId,
    url: c.req.url,
  })
})
```

### Optional Parameters

```ts
// Matches both /api/animals and /api/animals/dogs
app.get('/api/animals/:type?', (c) => {
  const type = c.req.param('type')

  if (type) {
    return c.text(`Showing ${type}`)
  }
  return c.text('Showing all animals')
})
```

## Wildcard and Pattern Examples

### Basic Wildcards

```ts
// Matches /static/any/path/file.css
app.get('/static/*', (c) => {
  const path = c.req.path
  return c.text(`Serving static file: ${path}`)
})

// Nested wildcards
app.get('/api/*/data/*', (c) => {
  return c.text(`API data endpoint: ${c.req.path}`)
})
```

### Regular Expression Patterns

```ts
// Date format validation: /posts/20240101/my-title
app.get('/posts/:date{[0-9]+}/:title{[a-z-]+}', async (c) => {
  const { date, title } = c.req.param()

  // Validate date format
  if (date.length !== 8) {
    return c.text('Invalid date format', 400)
  }

  return c.json({
    date,
    title: title.replace(/-/g, ' '),
  })
})
```

### File Extensions

```ts
// Match PNG files with paths: /images/photo.png, /images/folder/photo.png
app.get('/images/:filename{.+\\.png}', async (c) => {
  const filename = c.req.param('filename')

  return c.json({
    message: `Serving PNG: ${filename}`,
    type: 'image/png',
  })
})
```

## Route Chaining Examples

### Method Chaining

```ts
app
  .get('/users', (c) => c.json({ users: [] }))
  .post((c) => c.json({ message: 'User created' }, 201))
  .put('/:id', (c) =>
    c.json({ message: `User ${c.req.param('id')} updated` })
  )
  .delete('/:id', (c) => c.text('', 204))
```

### Resource-Based Routing

```ts
// RESTful API pattern
app
  .get('/posts', (c) => c.json({ posts: [] }))
  .post('/posts', async (c) => {
    const body = await c.req.json()
    return c.json({ id: 123, ...body }, 201)
  })
  .get('/posts/:id', (c) => c.json({ id: c.req.param('id') }))
  .put('/posts/:id', async (c) => {
    const body = await c.req.json()
    return c.json({ id: c.req.param('id'), ...body })
  })
  .delete('/posts/:id', (c) => c.text('', 204))
```

## Route Grouping Examples

### Basic Grouping

```ts
import { Hono } from 'hono'

// Create a group for book-related routes
const bookRoutes = new Hono()

bookRoutes.get('/', (c) => c.json({ books: [] })) // GET /books
bookRoutes.get('/:id', (c) => {
  // GET /books/:id
  const id = c.req.param('id')
  return c.json({ book: { id, title: `Book ${id}` } })
})
bookRoutes.post('/', (c) => c.json({ message: 'Book created' }, 201)) // POST /books

// Mount the group
const app = new Hono()
app.route('/books', bookRoutes)
```

### Nested Grouping

```ts
// API version grouping
const v1 = new Hono()
v1.get('/users', (c) => c.json({ version: '1.0', users: [] }))

const v2 = new Hono()
v2.get('/users', (c) => c.json({ version: '2.0', users: [] }))

const api = new Hono()
api.route('/v1', v1)
api.route('/v2', v2)

const app = new Hono()
app.route('/api', api)

// Results in: /api/v1/users and /api/v2/users
```

### Grouping Without Changing Base

```ts
const userRoutes = new Hono()
userRoutes.get('/users', (c) => c.json({ users: [] }))
userRoutes.post('/users', (c) => c.json({ message: 'User created' }))

const postRoutes = new Hono().basePath('/posts')
postRoutes.get('/', (c) => c.json({ posts: [] })) // /posts
postRoutes.post('/', (c) => c.json({ message: 'Post created' })) // /posts

const app = new Hono()
app.route('/', userRoutes) // Handles /users
app.route('/', postRoutes) // Handles /posts
```

## Base Path Examples

### API Versioning

```ts
const apiV1 = new Hono().basePath('/api/v1')

apiV1.get('/users', (c) => c.json({ users: [] })) // /api/v1/users
apiV1.get('/posts', (c) => c.json({ posts: [] })) // /api/v1/posts

const app = new Hono()
app.route('/', apiV1)
```

### Microservice Pattern

```ts
const authService = new Hono().basePath('/auth')
authService.post('/login', (c) => c.json({ token: 'abc123' }))
authService.post('/logout', (c) => c.text('', 204))

const userService = new Hono().basePath('/users')
userService.get('/', (c) => c.json({ users: [] }))
userService.get('/:id', (c) => c.json({ user: {} }))

const app = new Hono()
app.route('/', authService) // /auth/login, /auth/logout
app.route('/', userService) // /users, /users/:id
```

## Advanced Routing Examples

### Hostname-Based Routing

```ts
const app = new Hono({
  getPath: (req) => req.url.replace(/^https?:\/\/([^?]+).*$/, '$1'),
})

app.get('/api.example.com/data', (c) =>
  c.json({ api: 'v1', data: [] })
)

app.get('/admin.example.com/dashboard', (c) =>
  c.json({ dashboard: 'admin' })
)
```

### Host Header Routing

```ts
const app = new Hono({
  getPath: (req) =>
    '/' +
    req.headers.get('host') +
    req.url.replace(/^https?:\/\/[^/]+(\/[^?]*).*/, '$1'),
})

app.get('/api.example.com/users', (c) =>
  c.json({ users: [], host: 'api' })
)

// Match: new Request('http://api.example.com/users', {
//   headers: { host: 'api.example.com' }
// })
```

### User-Agent Based Routing

```ts
const app = new Hono({
  getPath: (req) => {
    const userAgent = req.headers.get('user-agent') || ''
    const isMobile = /Mobile|Android|iPhone/.test(userAgent)
    const prefix = isMobile ? '/mobile' : '/desktop'

    return prefix + new URL(req.url).pathname
  },
})

app.get('/mobile/app', (c) => c.text('Mobile app'))
app.get('/desktop/app', (c) => c.text('Desktop app'))
```

## Route Priority Examples

### Specific Before General

```ts
// ✅ Correct order - specific routes first
app.get('/posts/recent', (c) => c.json({ recent: true }))
app.get('/posts/:id', (c) => c.json({ id: c.req.param('id') }))

// GET /posts/recent -> { recent: true }
// GET /posts/123 -> { id: "123" }
```

### Middleware Execution Order

```ts
// Middleware runs in registration order
app.use('*', async (c, next) => {
  console.log('First middleware')
  await next()
})

app.use('/api/*', async (c, next) => {
  console.log('API middleware')
  await next()
})

app.get('/api/users', (c) => {
  console.log('Handler')
  return c.json({ users: [] })
})

// GET /api/users logs: "First middleware", "API middleware", "Handler"
```

### Fallback Routes

```ts
// Specific routes first
app.get('/home', (c) => c.text('Home page'))
app.get('/about', (c) => c.text('About page'))

// Fallback route last
app.get('*', (c) => c.text('Page not found', 404))
```

## Group Ordering Examples

### Correct Grouping Order

```ts
const three = new Hono()
three.get('/hi', (c) => c.text('hi'))

const two = new Hono()
two.route('/three', three) // Add routes before mounting

const app = new Hono()
app.route('/two', two)

// ✅ GET /two/three/hi -> "hi"
```

### Common Grouping Mistake

```ts
const three = new Hono()
three.get('/hi', (c) => c.text('hi'))

const two = new Hono()

const app = new Hono()
app.route('/two', two) // ❌ Mounting empty group first
two.route('/three', three) // Routes added after mounting

// ❌ GET /two/three/hi -> 404 Not Found
```

## Complex Routing Scenarios

### API with Authentication Zones

```ts
const publicAPI = new Hono()
publicAPI.get('/health', (c) => c.json({ status: 'ok' }))
publicAPI.get('/info', (c) => c.json({ version: '1.0' }))

const protectedAPI = new Hono()
protectedAPI.use('*', async (c, next) => {
  const auth = c.req.header('Authorization')
  if (!auth?.startsWith('Bearer ')) {
    return c.text('Unauthorized', 401)
  }
  await next()
})
protectedAPI.get('/profile', (c) => c.json({ user: 'authenticated' }))
protectedAPI.get('/settings', (c) => c.json({ settings: {} }))

const app = new Hono()
app.route('/api/public', publicAPI)
app.route('/api/protected', protectedAPI)
```

### Multi-Version API

```ts
const createUserAPI = (version: string) => {
  const api = new Hono()

  api.get('/users', (c) =>
    c.json({
      version,
      users: [],
    })
  )

  api.get('/users/:id', (c) =>
    c.json({
      version,
      user: { id: c.req.param('id') },
    })
  )

  return api
}

const app = new Hono()
app.route('/v1', createUserAPI('1.0'))
app.route('/v2', createUserAPI('2.0'))
app.route('/v3', createUserAPI('3.0'))
```

### Resource Nesting

```ts
// Blog with posts and comments
const commentRoutes = new Hono()
commentRoutes.get('/', (c) => {
  const postId = c.req.param('postId')
  return c.json({ postId, comments: [] })
})
commentRoutes.post('/', async (c) => {
  const postId = c.req.param('postId')
  const body = await c.req.json()
  return c.json({ postId, comment: body }, 201)
})

const postRoutes = new Hono()
postRoutes.get('/', (c) => c.json({ posts: [] }))
postRoutes.get('/:id', (c) =>
  c.json({
    post: { id: c.req.param('id') },
  })
)
postRoutes.route('/:postId/comments', commentRoutes)

const app = new Hono()
app.route('/posts', postRoutes)

// Available routes:
// GET /posts
// GET /posts/:id
// GET /posts/:postId/comments
// POST /posts/:postId/comments
```

## Dynamic Routing Examples

### Content-Type Based Routing

```ts
// Same endpoint, different responses based on Accept header
app.get('/data', (c) => {
  const accept = c.req.header('Accept')

  if (accept?.includes('application/json')) {
    return c.json({ data: [1, 2, 3] })
  } else if (accept?.includes('text/csv')) {
    return c.text('1,2,3', 200, {
      'Content-Type': 'text/csv',
    })
  } else if (accept?.includes('application/xml')) {
    return c.text('<data>1,2,3</data>', 200, {
      'Content-Type': 'application/xml',
    })
  }

  return c.json({ data: [1, 2, 3] }) // Default to JSON
})
```

### Language-Based Routing

```ts
// Routes based on Accept-Language header
app.get('/welcome', (c) => {
  const acceptLanguage = c.req.header('Accept-Language') || ''

  if (acceptLanguage.includes('es')) {
    return c.text('¡Bienvenido!')
  } else if (acceptLanguage.includes('fr')) {
    return c.text('Bienvenue!')
  } else if (acceptLanguage.includes('ja')) {
    return c.text('いらっしゃいませ!')
  }

  return c.text('Welcome!')
})
```

## Error Handling in Routes

### Route-Specific Error Handling

```ts
import { HTTPException } from 'hono/http-exception'

app.get('/users/:id', async (c) => {
  const id = c.req.param('id')

  // Validate ID format
  if (!/^\d+$/.test(id)) {
    throw new HTTPException(400, {
      message: 'Invalid user ID format',
    })
  }

  // Simulate user lookup
  const user = await findUser(parseInt(id))
  if (!user) {
    throw new HTTPException(404, {
      message: 'User not found',
    })
  }

  return c.json({ user })
})

async function findUser(id: number) {
  // Mock user lookup
  return id === 1 ? { id: 1, name: 'Alice' } : null
}
```

### Graceful Degradation

```ts
app.get('/search/:term?', async (c) => {
  const term = c.req.param('term')

  if (!term) {
    return c.json({
      message: 'No search term provided',
      suggestions: ['javascript', 'typescript', 'hono'],
    })
  }

  // Simulate search
  const results = await searchFor(term)

  return c.json({
    term,
    results,
    count: results.length,
  })
})

async function searchFor(term: string) {
  // Mock search
  return term.length > 2 ? [`Result for ${term}`] : []
}
```

## Middleware Integration Examples

### Route-Specific Middleware

```ts
import { cors } from 'hono/cors'
import { logger } from 'hono/logger'

const app = new Hono()

// Global middleware
app.use('*', logger())

// API-specific middleware
app.use(
  '/api/*',
  cors({
    origin: ['https://example.com'],
    allowMethods: ['GET', 'POST', 'PUT', 'DELETE'],
  })
)

// Auth middleware for admin routes
app.use('/admin/*', async (c, next) => {
  const token = c.req.header('Authorization')
  if (!isValidAdminToken(token)) {
    return c.text('Forbidden', 403)
  }
  await next()
})

app.get('/api/users', (c) => c.json({ users: [] }))
app.get('/admin/dashboard', (c) => c.text('Admin Dashboard'))

function isValidAdminToken(token: string | undefined): boolean {
  return token === 'Bearer admin-secret'
}
```

### Conditional Middleware

```ts
app.use('/upload/*', async (c, next) => {
  const contentType = c.req.header('Content-Type')

  if (!contentType?.startsWith('multipart/form-data')) {
    return c.text('Only multipart/form-data allowed', 400)
  }

  await next()
})

app.post('/upload/image', async (c) => {
  const body = await c.req.parseBody()
  const file = body['image'] as File

  return c.json({
    filename: file.name,
    size: file.size,
  })
})
```

## Performance Optimization Examples

### Route Grouping for Performance

```ts
// Group similar routes to take advantage of router optimizations
const apiRoutes = new Hono()

// User endpoints
apiRoutes.get('/users', (c) => c.json({ users: [] }))
apiRoutes.post('/users', (c) => c.json({ message: 'Created' }))
apiRoutes.get('/users/:id', (c) => c.json({ user: {} }))

// Post endpoints
apiRoutes.get('/posts', (c) => c.json({ posts: [] }))
apiRoutes.post('/posts', (c) => c.json({ message: 'Created' }))
apiRoutes.get('/posts/:id', (c) => c.json({ post: {} }))

const app = new Hono()
app.route('/api', apiRoutes)
```

### Static Route Optimization

```ts
// Put static routes before dynamic ones for better performance
app.get('/health', (c) => c.text('OK'))
app.get('/version', (c) => c.json({ version: '1.0.0' }))
app.get('/favicon.ico', (c) => c.text('', 404))

// Dynamic routes after static ones
app.get('/users/:id', (c) => c.json({ user: {} }))
app.get('/:category/:slug', (c) => c.json({ content: {} }))
```

## Real-World Application Examples

### Blog API

```ts
const blogAPI = new Hono()

// Posts
blogAPI.get('/posts', async (c) => {
  const page = parseInt(c.req.query('page') || '1')
  const limit = parseInt(c.req.query('limit') || '10')

  return c.json({
    posts: [],
    pagination: { page, limit, total: 0 },
  })
})

blogAPI.get('/posts/:slug', (c) => {
  const slug = c.req.param('slug')
  return c.json({ post: { slug, title: `Post ${slug}` } })
})

// Categories
blogAPI.get('/categories', (c) => c.json({ categories: [] }))
blogAPI.get('/categories/:name/posts', (c) => {
  const category = c.req.param('name')
  return c.json({
    category,
    posts: [],
  })
})

const app = new Hono()
app.route('/blog', blogAPI)
```

### E-commerce API

```ts
const storeAPI = new Hono()

// Products
storeAPI.get('/products', (c) => {
  const { category, price_min, price_max } = c.req.query()
  return c.json({
    products: [],
    filters: { category, price_min, price_max },
  })
})

storeAPI.get('/products/:id', (c) => {
  const id = c.req.param('id')
  return c.json({ product: { id } })
})

// Shopping cart
storeAPI.get('/cart', (c) => c.json({ items: [] }))
storeAPI.post('/cart/items', async (c) => {
  const { productId, quantity } = await c.req.json()
  return c.json(
    {
      message: 'Item added to cart',
      productId,
      quantity,
    },
    201
  )
})

// Orders
storeAPI.get('/orders', (c) => c.json({ orders: [] }))
storeAPI.get('/orders/:id', (c) => {
  const orderId = c.req.param('id')
  return c.json({ order: { id: orderId } })
})

const app = new Hono()
app.route('/store', storeAPI)
```

## See Also

- [Routing API Specification](/docs/api/routing) - Complete routing API reference
- [Context Examples](/docs/api/context-examples) - Request handling patterns
- [Hono Examples](/docs/api/hono-examples) - Application setup patterns
- [Middleware Guide](/docs/guides/middleware) - Middleware implementation patterns
