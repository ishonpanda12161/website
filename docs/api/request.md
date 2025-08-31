# HonoRequest Specification

The `HonoRequest` class wraps the standard Web API `Request` object with additional convenience methods for common server operations.

## Type Definition

```ts
class HonoRequest<ParamKey extends string = string, Input = {}>
```

**Type Parameters:**
- `ParamKey`: Union type of path parameter names
- `Input`: Type schema for validated input data

**Access:** Available as `c.req` in request handlers and middleware

## Parameter Access Methods

### `param()`

Access path parameters captured from the route pattern.

#### `param(key: ParamKey): string | undefined`

Get a specific path parameter.

**Parameters:**
- `key`: `ParamKey` - Path parameter name from the route pattern

**Returns:** `string | undefined` - Parameter value (URL-decoded) or `undefined` if not found

#### `param(): Record<string, string>`

Get all path parameters as an object.

**Returns:** `Record<string, string>` - All parameters as key-value pairs

**Examples:**
- Route: `/user/:id` → `param('id')` returns the captured ID
- Route: `/posts/:postId/comment/:commentId` → `param()` returns both parameters

## Query String Methods

### `query()`

Access URL query parameters.

#### `query(key: string): string | undefined`

Get a specific query parameter.

**Parameters:**
- `key`: `string` - Query parameter name

**Returns:** `string | undefined` - Parameter value or `undefined` if not present

#### `query(): Record<string, string>`

Get all query parameters as an object.

**Returns:** `Record<string, string>` - All query parameters as key-value pairs

**Behavior:**
- Multiple values for same key: only the last value is returned
- Values are URL-decoded automatically

### `queries(key: string): string[] | undefined`

Get all values for a query parameter that appears multiple times.

**Parameters:**
- `key`: `string` - Query parameter name

**Returns:** `string[] | undefined` - Array of all values or `undefined` if not present

**Use Case:** URLs like `/search?tags=A&tags=B&tags=C`

## Header Access Methods

### `header()`

Access HTTP request headers.

#### `header(name: string): string | undefined`

Get a specific header value.

**Parameters:**
- `name`: `string` - Header name (case-insensitive)

**Returns:** `string | undefined` - Header value or `undefined` if not present

#### `header(): Record<string, string>`

Get all headers as an object.

**Returns:** `Record<string, string>` - All headers with lowercase keys

**Important:** When called with no arguments, all keys in the returned object are lowercase.

## Body Parsing Methods

### `parseBody(options?)`

Parse form data (multipart/form-data or application/x-www-form-urlencoded).

#### `parseBody(): Promise<Record<string, string | File>>`

Basic form parsing.

**Returns:** `Promise<Record<string, string | File>>` - Parsed form data

#### `parseBody(options: ParseBodyOptions): Promise<Record<string, string | File | (string | File)[]>>`

Advanced form parsing with options.

**Parameters:**
- `options`: `ParseBodyOptions` - Parsing configuration

**Options:**
```ts
type ParseBodyOptions = {
  all?: boolean    // Handle multiple values for same key
  dot?: boolean    // Enable dot notation parsing
}
```

**Behaviors:**
- **Single file**: `body['field']` is `string | File`
- **Multiple files**: Use `body['field[]']` - always returns `(string | File)[]`
- **all: true**: Multiple values with same name become arrays
- **dot: true**: Parse dot notation (`obj.key` becomes nested object)

### `json<T = any>(): Promise<T>`

Parse JSON request body.

**Returns:** `Promise<T>` - Parsed JSON object

**Requirements:**
- Content-Type should be `application/json`
- Body must contain valid JSON

### `text(): Promise<string>`

Parse request body as plain text.

**Returns:** `Promise<string>` - Request body as string

**Requirements:**
- Content-Type should be `text/plain`

### `arrayBuffer(): Promise<ArrayBuffer>`

Parse request body as ArrayBuffer.

**Returns:** `Promise<ArrayBuffer>` - Binary data as ArrayBuffer

**Use Cases:**
- File uploads
- Binary data processing
- Raw data handling

### `blob(): Promise<Blob>`

Parse request body as Blob.

**Returns:** `Promise<Blob>` - Request body as Blob object

**Properties Available:**
- `blob.size`: Size in bytes
- `blob.type`: MIME type

### `formData(): Promise<FormData>`

Parse request body as FormData.

**Returns:** `Promise<FormData>` - Native FormData object

**Use Cases:**
- Direct FormData manipulation
- Access to FormData methods (entries(), keys(), values())

## Validation Methods

### `valid(target: ValidTargets): any`

Access validated data from validation middleware.

**Parameters:**
- `target`: `ValidTargets` - Data source to validate

**Valid Targets:**
- `'form'` - Form data validation
- `'json'` - JSON body validation
- `'query'` - Query parameter validation
- `'header'` - Header validation
- `'cookie'` - Cookie validation
- `'param'` - Path parameter validation

**Returns:** Validated data with appropriate typing based on validation schema

**Requirements:**
- Validation middleware must be applied before using this method
- See [Validation Guide](/docs/guides/validation) for setup details

## Request Metadata Properties

### `path: string`

The pathname portion of the request URL.

**Example:** For URL `https://example.com/api/users?page=1`, `path` is `/api/users`

### `url: string`

The complete request URL.

**Example:** `https://example.com/api/users?page=1`

### `method: string`

The HTTP method of the request.

**Example:** `'GET'`, `'POST'`, `'PUT'`, `'DELETE'`, etc.

### `raw: Request`

The underlying Web API Request object.

**Use Cases:**
- Platform-specific features (e.g., Cloudflare Workers `cf` object)
- Direct access to Request methods not wrapped by HonoRequest
- Advanced request inspection

## Deprecated Properties

### `routePath` *(Deprecated)*

**Status:** Deprecated in v4.8.0 - Use `routePath()` from [Route Helper](/docs/helpers/route)

The registered route pattern for the current route.

### `matchedRoutes` *(Deprecated)*

**Status:** Deprecated in v4.8.0 - Use `matchedRoutes()` from [Route Helper](/docs/helpers/route)

Array of all matched routes for debugging purposes.

## Type Safety

### Parameter Typing

When using TypeScript with typed routes:

```ts
app.get('/user/:id', (c) => {
  const id = c.req.param('id') // Type: string | undefined
  //    ^? string | undefined
})
```

### Input Validation Typing

With validation middleware:

```ts
type UserInput = {
  name: string
  email: string
}

app.post('/user', validator('json', schema), (c) => {
  const { name, email } = c.req.valid('json') // Type: UserInput
  //      ^? UserInput
})
```

## Error Handling

### Parsing Errors

All body parsing methods can throw errors:

- **JSON parsing**: Invalid JSON syntax
- **Form parsing**: Malformed form data
- **Binary parsing**: Stream reading errors

### Best Practices

```ts
app.post('/api', async (c) => {
  try {
    const body = await c.req.json()
    // Process body
  } catch (error) {
    return c.json({ error: 'Invalid JSON' }, 400)
  }
})
```

## Performance Considerations

- **Body parsing**: Each parsing method consumes the request body stream
- **Multiple calls**: Don't call multiple body parsing methods on the same request
- **Large uploads**: Consider streaming for large file uploads
- **Header access**: `header()` with specific name is more efficient than `header()`

## See Also

- [HonoRequest Examples](/docs/api/request-examples) - Practical usage examples and patterns
- [Context Specification](/docs/api/context) - Context object containing HonoRequest
- [Validation Guide](/docs/guides/validation) - Request validation patterns
- [Routing Specification](/docs/api/routing) - Route patterns and parameter capture