# Routing Examples

Practical examples and patterns for Hono routing functionality.

## Basic Routing Examples

### HTTP Methods

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// HTTP Methods
app.get('/', (c) => c.text('GET /'))
app.post('/', (c) => c.text('POST /'))
app.put('/', (c) => c.text('PUT /'))
app.delete('/', (c) => c.text('DELETE /'))

// Wildcard
app.get('/wild/*/card', (c) => {
  return c.text('GET /wild/*/card')
})

// Any HTTP methods
app.all('/hello', (c) => c.text('Any Method /hello'))

// Custom HTTP method
app.on('PURGE', '/cache', (c) => c.text('PURGE Method /cache'))

// Multiple Method
app.on(['PUT', 'DELETE'], '/post', (c) =>
  c.text('PUT or DELETE /post')
)

// Multiple Paths
app.on('GET', ['/hello', '/ja/hello', '/en/hello'], (c) =>
  c.text('Hello')
)
```

## Path Parameter Examples

### Basic Parameters

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/user/:name', async (c) => {
  const name = c.req.param('name')
  //       ^?
  // ...
})
```

### Multiple Parameters

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/posts/:id/comment/:comment_id', async (c) => {
  const { id, comment_id } = c.req.param()
  //       ^?
  // ...
})
```

### Optional Parameters

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
// Will match `/api/animal` and `/api/animal/:type`
app.get('/api/animal/:type?', (c) => c.text('Animal!'))
```

## Advanced Pattern Examples

### Regular Expression Patterns

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/post/:date{[0-9]+}/:title{[a-z]+}', async (c) => {
  const { date, title } = c.req.param()
  //       ^?
  // ...
})
```

### Including Slashes in Parameters

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/posts/:filename{.+\\.png}', async (c) => {
  //...
})
```

## Method Chaining Examples

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

## Route Grouping Examples

### Basic Grouping

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

### Grouping Without Changing Base

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

## Base Path Examples

```ts twoslash
import { Hono } from 'hono'
// ---cut---
const api = new Hono().basePath('/api')
api.get('/book', (c) => c.text('List Books')) // GET /api/book
```

## Hostname-Based Routing Examples

### Including Hostname in Routes

```ts twoslash
import { Hono } from 'hono'
// ---cut---
const app = new Hono({
  getPath: (req) => req.url.replace(/^https?:\/([^?]+).*$/, '$1'),
})

app.get('/www1.example.com/hello', (c) => c.text('hello www1'))
app.get('/www2.example.com/hello', (c) => c.text('hello www2'))
```

### Using Host Header Value

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

// A following request will match the route:
// new Request('http://www1.example.com/hello', {
//  headers: { host: 'www1.example.com' },
// })
```

## Routing Priority Examples

### Registration Order

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/book/a', (c) => c.text('a')) // a
app.get('/book/:slug', (c) => c.text('common')) // common
```

**Result:**
```
GET /book/a ---> `a`
GET /book/b ---> `common`
```

### Handler Execution Stopping

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('*', (c) => c.text('common')) // common
app.get('/foo', (c) => c.text('foo')) // foo
```

**Result:**
```
GET /foo ---> `common` // foo will not be dispatched
```

### Middleware vs Handler Order

```ts twoslash
import { Hono } from 'hono'
import { logger } from 'hono/logger'
const app = new Hono()
// ---cut---
app.use(logger())
app.get('/foo', (c) => c.text('foo'))
```

### Fallback Handler

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.get('/bar', (c) => c.text('bar')) // bar
app.get('*', (c) => c.text('fallback')) // fallback
```

**Result:**
```
GET /bar ---> `bar`
GET /foo ---> `fallback`
```

## Route Grouping Order Examples

### Correct Order

```ts
three.get('/hi', (c) => c.text('hi'))
two.route('/three', three)
app.route('/two', two)

export default app
```

**Result:**
```
GET /two/three/hi ---> `hi`
```

### Incorrect Order (404 Result)

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
const two = new Hono()
const three = new Hono()
// ---cut---
three.get('/hi', (c) => c.text('hi'))
app.route('/two', two) // `two` does not have routes
two.route('/three', three)

export default app
```

**Result:**
```
GET /two/three/hi ---> 404 Not Found
```

## See Also

- [Routing Specification](/docs/api/routing) - Complete formal routing specification
- [Hono Examples](/docs/api/hono-examples) - Application setup and configuration examples
- [Context Examples](/docs/api/context-examples) - Request-response handling patterns