# Routing Specification

This document provides a comprehensive specification for Hono's routing system, including path patterns, matching rules, and priority behavior.

## Path Pattern Syntax

### Static Paths

Static paths match exactly as written.

**Format:** `/exact/path`

**Matching Rules:**
- Exact string comparison
- Case-sensitive matching
- No parameter capture

**Examples:**
- `/` - Root path
- `/users` - Static path segment
- `/api/v1/status` - Multi-segment static path

### Named Parameters

Named parameters capture path segments for dynamic routing.

**Format:** `/:paramName`

**Type:** `string` (captured segment value)

**Matching Rules:**
- Matches any non-empty path segment
- Captured value is URL-decoded
- Accessible via `c.req.param(paramName)`

**Constraints:**
- Parameter names must be valid JavaScript identifiers
- Cannot contain `/` character
- Cannot be empty

### Optional Parameters

Parameters that may or may not be present in the path.

**Format:** `/:paramName?`

**Matching Rules:**
- Matches path with or without the parameter segment
- Parameter value is `undefined` when segment is missing
- Trailing segments after optional parameter still must match

**Behavior:**
- `/api/animal/:type?` matches both `/api/animal` and `/api/animal/dog`
- When missing, `c.req.param('type')` returns `undefined`

### Wildcard Paths

Wildcard segments match any single path segment.

**Format:** `/path/*/segment`

**Matching Rules:**
- `*` matches exactly one non-empty path segment
- Does not capture the matched value
- Cannot be accessed via `c.req.param()`

**Limitations:**
- Cannot appear at the end of a path pattern
- Cannot be used within a segment (e.g., `/*abc` is invalid)

### Regular Expression Constraints

Parameters can be constrained using regular expressions.

**Format:** `/:paramName{regexPattern}`

**Matching Rules:**
- Parameter must match the provided regular expression
- Full parameter value must match (anchored match)
- Standard JavaScript RegExp syntax

**Examples:**
- `/:id{[0-9]+}` - Numeric ID only
- `/:slug{[a-zA-Z0-9-]+}` - Alphanumeric with hyphens
- `/:filename{.+\\.png}` - Filename ending with .png (may include slashes)

**Notes:**
- Regex patterns are not automatically anchored
- Use `^` and `$` if needed, though full match is enforced
- Complex patterns may impact routing performance

## Route Matching Algorithm

### Matching Process

1. **Path Extraction**: Extract path from request URL using `getPath` function
2. **Route Iteration**: Test routes in registration order
3. **Pattern Matching**: Apply pattern matching rules for each route
4. **First Match Wins**: Execute first matching route's handlers

### Priority Rules

**Registration Order Priority:**
Routes are matched in the order they were registered (first match wins).

**Pattern Specificity:**
When routes are registered in the same order:
1. Static paths (exact matches)
2. Constrained parameters (regex patterns)  
3. Named parameters (unconstrained)
4. Optional parameters
5. Wildcard patterns

### Trailing Slash Behavior

Controlled by the `strict` constructor option:

**Strict Mode (`strict: true` - default):**
- `/hello` and `/hello/` are different routes
- Must match exactly as registered
- No automatic normalization

**Non-Strict Mode (`strict: false`):**
- `/hello` and `/hello/` are treated as equivalent
- Automatic trailing slash normalization
- More flexible matching

## Route Registration Methods

### HTTP Method Registration

All HTTP method functions follow the same signature:

**Signature:**
```ts
get|post|put|delete|head|options|patch(
  path: string,
  ...handlers: Handler[]
): Hono
```

**Behavior:**
- Registers route for specific HTTP method
- Supports method chaining
- Multiple handlers execute in order

### Universal Method Registration

#### `all(path, ...handlers)`

**Signature:**
```ts
all(path: string, ...handlers: Handler[]): Hono
```

**Behavior:**
- Matches any HTTP method for the given path
- Executes for GET, POST, PUT, DELETE, etc.
- Useful for universal middleware or handlers

#### `on(method, path, ...handlers)`

**Signature:**
```ts
on(
  method: string | string[],
  path: string | string[],
  ...handlers: Handler[]
): Hono
```

**Behavior:**
- Custom method(s) and path(s) registration
- Supports arrays for multiple methods or paths
- Cartesian product matching (each method × each path)

## Route Group Management

### Sub-Application Mounting

#### `route(path, app)`

**Signature:**
```ts
route(path: string, app: Hono): Hono
```

**Behavior:**
- Mounts all routes from `app` under `path` prefix
- Preserves middleware and error handlers from sub-app
- Sub-app routes become relative to mount path

**Requirements:**
- `path` must start with `/`
- `app` must be a valid Hono instance
- Sub-app should be fully configured before mounting

### Base Path Configuration

#### `basePath(path)`

**Signature:**
```ts
basePath(path: string): Hono
```

**Behavior:**
- Returns new Hono instance with base path applied
- All subsequent routes are automatically prefixed
- Original instance remains unchanged

**Requirements:**
- `path` must start with `/`
- `path` must not end with `/` (except for root `/`)

## Custom Path Extraction

### `getPath` Function

**Signature:**
```ts
getPath?: (request: Request) => string
```

**Purpose:**
- Override default URL pathname extraction
- Enable hostname-based routing
- Support custom routing logic

**Requirements:**
- Must return a valid path string starting with `/`
- Should handle all expected request formats
- Performance-critical (called for every request)

**Common Patterns:**
- Hostname integration: Include hostname in routing path
- Header-based routing: Use headers for route determination
- Protocol handling: Different routing based on HTTP vs HTTPS

## Performance Characteristics

### Router-Specific Behavior

Different routers have different performance characteristics for pattern matching:

- **RegExpRouter**: O(1) for simple patterns, compiled to single regex
- **TrieRouter**: O(log n) lookup with tree traversal
- **LinearRouter**: O(n) linear search through all routes
- **PatternRouter**: O(n) simplified pattern matching
- **SmartRouter**: Automatically selects optimal router

### Optimization Guidelines

1. **Register specific routes first**: Static paths before parameterized
2. **Minimize regex complexity**: Simple patterns perform better
3. **Use appropriate router**: Match router to route complexity
4. **Limit route count**: Fewer routes generally perform better

## Caveats and Limitations

### Pattern Limitations

- **No partial wildcards**: Patterns like `/file*.txt` are not supported
- **No nested parameters**: Patterns like `/:a/:b{:c}` are invalid
- **Case sensitivity**: All matching is case-sensitive

### Execution Behavior

- **First match wins**: Later routes with same pattern are unreachable
- **Handler stopping**: Returned Response stops middleware chain
- **Error propagation**: Errors bubble up to global error handler

### Route Order Dependencies

Incorrect registration order can make routes unreachable:

```ts
// WRONG: Generic route registered first
app.get('/:id', handler)      // This will catch everything
app.get('/special', handler)  // This will never execute

// CORRECT: Specific routes first  
app.get('/special', handler)  // This executes for /special
app.get('/:id', handler)      // This executes for other paths
```

## See Also

- [Routing Examples](/docs/api/routing-examples) - Practical routing patterns and examples
- [Hono Specification](/docs/api/hono) - Main application class specification
- [Context Specification](/docs/api/context) - Request-response context object
- [Router Presets](/docs/api/presets) - Available router configurations
