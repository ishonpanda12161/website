# HonoRequest Specification

The `HonoRequest` object is an enhanced wrapper around the Web Standards [`Request`](https://developer.mozilla.org/en-US/docs/Web/API/Request) object, providing convenient methods for accessing request data.

## Type Definition

```ts
class HonoRequest<P = {}, I extends Input = {}> extends Request
```

**Type Parameters:**

- `P`: Type for path parameters
- `I`: Type for input validation

**Access:** Available as `c.req` in request handlers and middleware

## Path Parameters

### `param(key?)`

Extract path parameters from the route.

**Overloads:**

```ts
param(): Record<string, string>
param(key: string): string | undefined
```

**Parameters:**

- `key` (optional): `string` - Specific parameter name

**Returns:**

- `Record<string, string>` - All parameters when no key provided
- `string | undefined` - Specific parameter value when key provided

**Notes:**

- Parameters are extracted from route patterns like `/users/:id`
- Returns `undefined` if the specified key doesn't exist

## Query String

### `query(key?)`

Extract query string parameters.

**Overloads:**

```ts
query(): Record<string, string>
query(key: string): string | undefined
```

**Parameters:**

- `key` (optional): `string` - Specific query parameter name

**Returns:**

- `Record<string, string>` - All query parameters when no key provided
- `string | undefined` - Specific parameter value when key provided

**Notes:**

- Parses the URL query string
- Only returns the first value for duplicate keys

### `queries(key)`

Get multiple values for a query parameter.

**Parameters:**

- `key`: `string` - Query parameter name

**Returns:** `string[]` - Array of all values for the specified key

**Notes:**

- Useful for handling multiple values like `?tags=a&tags=b`
- Returns empty array if key doesn't exist

## Headers

### `header(name?)`

Access request headers.

**Overloads:**

```ts
header(): Record<string, string>
header(name: string): string | undefined
```

**Parameters:**

- `name` (optional): `string` - Specific header name

**Returns:**

- `Record<string, string>` - All headers when no name provided (keys are lowercase)
- `string | undefined` - Specific header value when name provided

**Notes:**

- Header names are case-insensitive when accessing specific headers
- When getting all headers, keys are normalized to lowercase

## Request Body Parsing

### `parseBody(options?)`

Parse form data from request body.

**Parameters:**

- `options` (optional): `ParseBodyOptions`
  - `all?: boolean` - Parse multiple values with same name as arrays
  - `dot?: boolean` - Parse dot notation into nested objects

**Returns:** `Promise<Record<string, string | File | (string | File)[]>>`

**Supported Content Types:**

- `multipart/form-data`
- `application/x-www-form-urlencoded`

**Notes:**

- File uploads are returned as `File` objects
- For multiple files with same name, use `name[]` notation or `all: true` option
- Dot notation creates nested objects when `dot: true`

### `json<T>()`

Parse JSON request body.

**Type Parameters:**

- `T`: Expected JSON structure type

**Returns:** `Promise<T>`

**Throws:** `SyntaxError` if JSON is invalid

**Notes:**

- Content-Type should be `application/json`
- Type parameter provides compile-time type safety

### `text()`

Parse text request body.

**Returns:** `Promise<string>`

**Notes:**

- Reads the entire request body as a string
- Suitable for `text/plain` content type

### `arrayBuffer()`

Parse request body as ArrayBuffer.

**Returns:** `Promise<ArrayBuffer>`

**Notes:**

- Returns raw binary data
- Useful for binary file processing

### `blob()`

Parse request body as Blob.

**Returns:** `Promise<Blob>`

**Notes:**

- Provides access to blob size and type
- Useful for file upload processing

### `formData()`

Parse request body as FormData.

**Returns:** `Promise<FormData>`

**Notes:**

- Direct access to FormData API
- Alternative to `parseBody()` for more control

## Validation

### `valid(target)`

Get validated data from request.

**Parameters:**

- `target`: `'form' | 'json' | 'query' | 'header' | 'cookie' | 'param'` - Validation target

**Returns:** Validated data with inferred types

**Notes:**

- Requires validation middleware to be applied first
- Type safety depends on validator configuration
- See [Validation Guide](/docs/guides/validation) for usage

## Request Properties

### `path`

The request pathname without query string.

**Type:** `string`

**Example:** For `http://example.com/users?page=1`, returns `/users`

### `url`

The complete request URL.

**Type:** `string`

**Example:** `http://example.com/users?page=1`

### `method`

The HTTP method of the request.

**Type:** `string`

**Example:** `GET`, `POST`, `PUT`, `DELETE`

### `raw`

The underlying Web Standards Request object.

**Type:** `Request`

**Notes:**

- Access to platform-specific request properties
- Use when HonoRequest methods are insufficient

## Legacy Properties

### `routePath`

**Status:** Deprecated in v4.8.0 - Use `routePath()` from [Route Helper](/docs/helpers/route) instead

The registered route pattern that matched this request.

**Type:** `string`

### `matchedRoutes`

**Status:** Deprecated in v4.8.0 - Use `matchedRoutes()` from [Route Helper](/docs/helpers/route) instead

Array of routes that matched during request processing.

**Type:** `MatchedRoute[]`

## Error Handling

Request parsing methods may throw errors:

- `json()` throws `SyntaxError` for invalid JSON
- `parseBody()` may throw for malformed form data
- Always handle these errors appropriately in your application

## Type Safety

The HonoRequest object integrates with Hono's type system:

```ts
// Path parameters are typed based on route pattern
app.get('/users/:id', (c) => {
  const id = c.req.param('id') // string | undefined
  // ...
})

// Validation provides strong typing
app.post('/users', zValidator('json', schema), (c) => {
  const user = c.req.valid('json') // Fully typed based on schema
  // ...
})
```

## See Also

- [Context Specification](/docs/api/context) - Parent context object
- [Routing Specification](/docs/api/routing) - Path parameter extraction
- [Request Examples](/docs/api/request-examples) - Practical usage patterns
- [Validation Guide](/docs/guides/validation) - Request validation patterns
