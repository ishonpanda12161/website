# HonoRequest Specification

The `HonoRequest` class wraps the standard [Request](https://developer.mozilla.org/en-US/docs/Web/API/Request) object with additional parsing and utility methods.

## Type Definition

```ts
class HonoRequest<ParamKeyType extends string = string, InputType = unknown>
```

**Type Parameters:**
- `ParamKeyType`: Union type of available path parameter keys
- `InputType`: Type for validated input data

## Path Parameters

### `.param(key?)`

Access path parameters from dynamic routes.

**Signature:**
```ts
param(): Record<string, string>
param<Key extends ParamKeyType>(key: Key): string | undefined
```

**Parameters:**
- `key` (optional): `string` - Parameter name to retrieve

**Returns:**
- When `key` provided: `string | undefined` - Parameter value or undefined
- When no `key`: `Record<string, string>` - All parameters as object

**Path Pattern Matching:**
- `:name` - Captures path segment as parameter
- `:name?` - Optional parameter
- `:name{regex}` - Parameter with regex constraint

## Query Parameters

### `.query(key?)`

Access URL query string parameters.

**Signature:**
```ts
query(): Record<string, string>
query(key: string): string | undefined
```

**Parameters:**
- `key` (optional): `string` - Query parameter name

**Returns:**
- When `key` provided: `string | undefined` - First value for parameter
- When no `key`: `Record<string, string>` - All parameters as object

**Note:** Returns only the first value for parameters with multiple values.

### `.queries(key?)`

Access all values for query parameters (including multiple values).

**Signature:**
```ts
queries(): Record<string, string[]>
queries(key: string): string[] | undefined
```

**Parameters:**
- `key` (optional): `string` - Query parameter name

**Returns:**
- When `key` provided: `string[] | undefined` - All values for parameter
- When no `key`: `Record<string, string[]>` - All parameters with all values

## Headers

### `.header(name?)`

Access request headers.

**Signature:**
```ts
header(): Record<string, string>
header(name: string): string | undefined
```

**Parameters:**
- `name` (optional): `string` - Header name (case-insensitive)

**Returns:**
- When `name` provided: `string | undefined` - Header value
- When no `name`: `Record<string, string>` - All headers as object (keys are lowercase)

**Important:** When called without arguments, all keys in the returned record are lowercase.

## Body Parsing

### `.parseBody(options?)`

Parse request body based on Content-Type header.

**Signature:**
```ts
parseBody<T = Record<string, string | File>>(options?: ParseBodyOptions): Promise<T>
```

**ParseBodyOptions:**
```ts
interface ParseBodyOptions {
  all?: boolean
  dot?: boolean
}
```

**Parameters:**
- `options` (optional): `ParseBodyOptions` - Parsing configuration
  - `all`: Parse all values including multiple files/fields with same name
  - `dot`: Enable dot notation parsing for nested objects

**Returns:** `Promise<T>` - Parsed body data

**Supported Content Types:**
- `application/x-www-form-urlencoded`
- `multipart/form-data` (with File objects for uploads)

**File Handling:**
- Single file: `body['fieldName']` is `string | File`
- Multiple files: Use `body['fieldName[]']` for `(string | File)[]`
- With `all: true`: Multiple values become arrays automatically

### `.json<T>()`

Parse JSON request body.

**Signature:**
```ts
json<T = any>(): Promise<T>
```

**Returns:** `Promise<T>` - Parsed JSON data

**Content-Type:** `application/json`
**Throws:** `SyntaxError` for invalid JSON

### `.text()`

Parse text request body.

**Signature:**
```ts
text(): Promise<string>
```

**Returns:** `Promise<string>` - Request body as text

**Content-Type:** `text/plain`

### `.arrayBuffer()`

Parse request body as ArrayBuffer.

**Signature:**
```ts
arrayBuffer(): Promise<ArrayBuffer>
```

**Returns:** `Promise<ArrayBuffer>` - Request body as ArrayBuffer

### `.blob()`

Parse request body as Blob.

**Signature:**
```ts
blob(): Promise<Blob>
```

**Returns:** `Promise<Blob>` - Request body as Blob

### `.formData()`

Parse request body as FormData.

**Signature:**
```ts
formData(): Promise<FormData>
```

**Returns:** `Promise<FormData>` - Request body as FormData

**Content-Type:** `multipart/form-data` or `application/x-www-form-urlencoded`

## Validation

### `.valid(target)`

Get validated data from middleware validation.

**Signature:**
```ts
valid<T = InputType>(target: ValidTarget): T
```

**Parameters:**
- `target`: `ValidTarget` - Data source to validate

**ValidTarget:**
- `'form'` - Form data validation
- `'json'` - JSON body validation
- `'query'` - Query parameters validation
- `'header'` - Header validation
- `'cookie'` - Cookie validation
- `'param'` - Path parameters validation

**Returns:** `T` - Validated data

**Note:** Must be used after validation middleware. See [Validation Guide](/docs/guides/validation).

## Route Information

### `.routePath` *(deprecated)*

::: warning
**Deprecated in v4.8.0**: Use `routePath()` from [Route Helper](/docs/helpers/route) instead.
:::

The matched route pattern.

**Type:** `string`

**Description:** Returns the original route pattern that matched the request (e.g., `/users/:id`).

### `.matchedRoutes` *(deprecated)*

::: warning
**Deprecated in v4.8.0**: Use `matchedRoutes()` from [Route Helper](/docs/helpers/route) instead.
:::

All routes that matched the request.

**Type:** `MatchedRoute[]`

**MatchedRoute:**
```ts
interface MatchedRoute {
  path: string
  method: string
  handler: Handler
}
```

**Description:** Array of all route objects that matched the current request, useful for debugging routing.

## Request Properties

### `.path`

The request pathname without query string.

**Type:** `string`

**Description:** The pathname portion of the URL (e.g., `/api/users/123`).

### `.url`

The complete request URL.

**Type:** `string`

**Description:** Full URL including protocol, host, pathname, and query string.

### `.method`

The HTTP method of the request.

**Type:** `string`

**Description:** HTTP method in uppercase (e.g., `'GET'`, `'POST'`, `'PUT'`).

### `.raw`

The original Request object.

**Type:** `Request`

**Description:** Access to the underlying [Request](https://developer.mozilla.org/en-US/docs/Web/API/Request) object for platform-specific features.

## Platform-Specific Features

The `.raw` property provides access to platform-specific extensions:

**Cloudflare Workers:**
```ts
// Access Cloudflare-specific properties
const metadata = c.req.raw.cf?.hostMetadata
const country = c.req.raw.cf?.country
```

**Other Platforms:**
- Custom headers and properties added by the runtime
- Platform-specific request metadata
- Extended Request features

## Error Handling

All parsing methods can throw errors:

- **JSON parsing**: `SyntaxError` for invalid JSON
- **Form parsing**: `TypeError` for invalid form data
- **Missing data**: Methods return `undefined` for missing optional data

## See Also

- [Context Specification](/docs/api/context) - The context object containing HonoRequest
- [Routing Specification](/docs/api/routing) - Path parameter patterns
- [Validation Guide](/docs/guides/validation) - Input validation patterns
- [Request Examples](/docs/api/request-examples) - Practical usage examples