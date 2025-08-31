# HonoRequest Specification

The `HonoRequest` object wraps the standard [Request](https://developer.mozilla.org/en-US/docs/Web/API/Request) object and provides additional methods for working with path parameters, query strings, headers, and request body parsing.

## Type Definition

```ts
HonoRequest<ParamKeys = string>
```

Type parameters:
- ParamKeys — Union type of path parameter keys for type-safe parameter access.

## Path Parameters

### param()

Extract path parameters from the URL.

```ts
param(): Record<string, string>
param(key: string): string | undefined
```

Parameters:
- key (optional): string — Specific parameter key to extract.

Returns:
- If key provided: `string | undefined` — Parameter value or undefined.
- If no key: `Record<string, string>` — All path parameters.

## Query String Methods

### query()

Extract query string parameters.

```ts
query(): Record<string, string>
query(key: string): string | undefined
```

Parameters:
- key (optional): string — Specific query parameter key.

Returns:
- If key provided: `string | undefined` — Query parameter value or undefined.
- If no key: `Record<string, string>` — All query parameters.

### queries()

Get multiple query string parameter values (for parameters that appear multiple times).

```ts
queries(key: string): string[] | undefined
```

Parameters:
- key: string — Query parameter key to extract.

Returns: `string[] | undefined` — Array of values for the parameter, or undefined.

## Headers

### header()

Access request headers.

```ts
header(): Record<string, string>
header(name: string): string | undefined
```

Parameters:
- name (optional): string — Header name to retrieve.

Returns:
- If name provided: `string | undefined` — Header value or undefined.
- If no name: `Record<string, string>` — All headers (keys are lowercase).

**Caveat:** When called without arguments, all header keys are normalized to lowercase.

## Request Body Parsing

### parseBody()

Parse multipart/form-data or application/x-www-form-urlencoded request bodies.

```ts
parseBody(options?: ParseBodyOptions): Promise<Record<string, string | File>>
```

Parameters:
- options (optional): ParseBodyOptions
  - all: boolean — Parse multiple values with same name into arrays.
  - dot: boolean — Structure result using dot notation.

Returns: `Promise<Record<string, string | File>>`

**Behavior:**
- Single values: `body['key']` is `string | File`
- Multiple values (with `[]` suffix): `body['key[]']` is `(string | File)[]`
- With `all: true`: Multiple values without suffix become arrays when needed.

### json()

Parse JSON request body.

```ts
json<T = any>(): Promise<T>
```

Returns: `Promise<T>` — Parsed JSON data.

**Constraint:** Content-Type must be application/json.

### text()

Parse request body as plain text.

```ts
text(): Promise<string>
```

Returns: `Promise<string>` — Request body as string.

### arrayBuffer()

Parse request body as ArrayBuffer.

```ts
arrayBuffer(): Promise<ArrayBuffer>
```

Returns: `Promise<ArrayBuffer>` — Request body as ArrayBuffer.

### blob()

Parse request body as Blob.

```ts
blob(): Promise<Blob>
```

Returns: `Promise<Blob>` — Request body as Blob.

### formData()

Parse request body as FormData.

```ts
formData(): Promise<FormData>
```

Returns: `Promise<FormData>` — Native FormData object.

## Validation

### valid()

Get validated data from middleware processing.

```ts
valid(target: string): any
```

Parameters:
- target: string — Validation target ('form', 'json', 'query', 'header', 'cookie', 'param').

Returns: `any` — Validated data structure.

**Note:** Requires validation middleware to be set up first.

## Request Properties

### path

The pathname portion of the request URL.

Type: `string`

### url

The complete request URL.

Type: `string`

### method

The HTTP method of the request.

Type: `string`

### raw

The underlying Web Standard Request object.

Type: `Request`

**Use case:** Access platform-specific properties (e.g., `c.req.raw.cf` for Cloudflare).

## Deprecated Properties

### routePath

**Deprecated in v4.8.0:** Use `routePath()` from [Route Helper](/docs/helpers/route) instead.

Returns the registered route pattern.

Type: `string`

### matchedRoutes

**Deprecated in v4.8.0:** Use `matchedRoutes()` from [Route Helper](/docs/helpers/route) instead.

Returns array of matched routes for debugging.

Type: `MatchedRoute[]`

## See Also

- [Context](/docs/api/context) — The request context object
- [Routing](/docs/api/routing) — Path patterns and parameter syntax
- [Route Helper](/docs/helpers/route) — Helper functions for route information