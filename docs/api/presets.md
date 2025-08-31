# Router Presets Specification

Hono provides pre-configured router combinations optimized for different deployment scenarios and performance requirements.

## Overview

Router presets are pre-configured `Hono` class exports that use specific router implementations optimized for different environments. All presets use the same `Hono` class with identical APIs; only the underlying router implementation differs.

## Available Presets

### `hono` (Default)

**Import:**
```ts
import { Hono } from 'hono'
```

**Router Configuration:**
```ts
new SmartRouter({
  routers: [new RegExpRouter(), new TrieRouter()]
})
```

**Characteristics:**
- **Initialization**: Slower route registration due to multiple router setup
- **Runtime Performance**: High performance once initialized
- **Memory Usage**: Moderate memory footprint
- **Best For**: Long-running servers, environments where routes are registered once

**Optimal Platforms:**
- Node.js servers
- Deno applications  
- Bun servers
- Cloudflare Workers (routes persist across requests)
- Deno Deploy
- Other platforms with persistent isolates

### `hono/quick`

**Import:**
```ts
import { Hono } from 'hono/quick'
```

**Router Configuration:**
```ts
new SmartRouter({
  routers: [new LinearRouter(), new TrieRouter()]
})
```

**Characteristics:**
- **Initialization**: Very fast route registration
- **Runtime Performance**: Good performance for moderate route counts
- **Memory Usage**: Lower memory usage during initialization
- **Best For**: Applications initialized per request

**Optimal Platforms:**
- Fastly Compute (initializes per request)
- AWS Lambda (cold starts)
- Edge environments with frequent initialization

### `hono/tiny`

**Import:**
```ts
import { Hono } from 'hono/tiny'
```

**Router Configuration:**
```ts
new PatternRouter()
```

**Characteristics:**
- **Initialization**: Fastest route registration
- **Runtime Performance**: Basic but sufficient for simple routing
- **Memory Usage**: Minimal memory footprint
- **Bundle Size**: Smallest package size
- **Best For**: Resource-constrained environments, simple applications

**Optimal Platforms:**
- IoT devices
- Edge functions with size constraints
- Simple microservices
- Prototype applications

## Router Implementation Details

### SmartRouter

Combines multiple router strategies for optimal performance:
- Uses different routers based on route characteristics
- Automatically selects the best router for each route type
- Balances initialization cost with runtime performance

### Component Routers

#### RegExpRouter
- **Strength**: Excellent runtime performance for complex route patterns
- **Weakness**: Slower initialization with many routes
- **Best For**: Complex routing patterns, parameter constraints

#### TrieRouter  
- **Strength**: Fast lookup for static routes and simple patterns
- **Weakness**: Higher memory usage with many routes
- **Best For**: Many static routes, prefix-based routing

#### LinearRouter
- **Strength**: Minimal memory usage, fast initialization
- **Weakness**: O(n) lookup time with route count
- **Best For**: Small to medium number of routes

#### PatternRouter
- **Strength**: Minimal overhead, fastest initialization
- **Weakness**: Basic pattern matching only
- **Best For**: Simple routing requirements

## Performance Characteristics

### Route Registration (Initialization)

**Fastest to Slowest:**
1. `hono/tiny` - PatternRouter only
2. `hono/quick` - LinearRouter + TrieRouter  
3. `hono` - RegExpRouter + TrieRouter

### Runtime Performance

**Route Lookup Speed (varies by route complexity):**

**Simple static routes:**
- All presets perform similarly well

**Parameterized routes:**
- `hono` (default): Excellent
- `hono/quick`: Good
- `hono/tiny`: Basic

**Complex regex patterns:**
- `hono` (default): Excellent  
- `hono/quick`: Limited support
- `hono/tiny`: Not supported

### Memory Usage

**Lowest to Highest:**
1. `hono/tiny`: Minimal memory overhead
2. `hono/quick`: Moderate memory usage
3. `hono`: Higher memory usage for complex routing

## Selection Guidelines

### Use `hono` (default) when:
- Building long-running servers (Node.js, Deno, Bun)
- Using complex routing patterns or regex constraints
- Performance after initialization is critical
- Memory usage is not a primary concern
- Routes are registered once at startup

### Use `hono/quick` when:
- Applications are frequently initialized (Fastly Compute)
- Cold start performance is critical (AWS Lambda)
- Moderate routing complexity is sufficient
- Balance between initialization and runtime speed needed

### Use `hono/tiny` when:
- Working in resource-constrained environments
- Bundle size is critical
- Simple routing patterns only
- Minimal memory footprint required
- Building prototypes or simple microservices

## Platform-Specific Recommendations

### Cloudflare Workers
**Recommended**: `hono` (default)
**Reason**: Worker isolates persist between requests, benefiting from optimized runtime performance

### Fastly Compute  
**Recommended**: `hono/quick`
**Reason**: New instances created per request, fast initialization is crucial

### AWS Lambda
**Recommended**: `hono/quick` or `hono/tiny`
**Reason**: Cold starts benefit from fast initialization

### Node.js/Bun/Deno Servers
**Recommended**: `hono` (default)
**Reason**: Long-running processes benefit from runtime optimization

### Edge Functions (Vercel, Netlify)
**Recommended**: `hono/quick`
**Reason**: Balance of initialization speed and runtime performance

### IoT/Embedded Environments
**Recommended**: `hono/tiny`
**Reason**: Minimal resource usage

## Feature Compatibility

### Routing Features by Preset

| Feature | `hono` | `hono/quick` | `hono/tiny` |
|---------|---------|--------------|-------------|
| Static routes | ✅ | ✅ | ✅ |
| Path parameters | ✅ | ✅ | ✅ |
| Optional parameters | ✅ | ✅ | ✅ |
| Wildcard routes | ✅ | ✅ | ✅ |
| Regex constraints | ✅ | ⚠️ Limited | ❌ |
| Complex patterns | ✅ | ⚠️ Basic | ❌ |
| Route priorities | ✅ | ✅ | ✅ |

### API Compatibility

All presets provide identical APIs:
- Same method signatures
- Same middleware support  
- Same error handling
- Same context objects
- Same helper functions

## Migration Between Presets

### Zero-Code Migration

Switching between presets requires only changing the import statement:

```ts
// From default
import { Hono } from 'hono'

// To quick
import { Hono } from 'hono/quick'

// To tiny  
import { Hono } from 'hono/tiny'
```

### Considerations When Switching

#### From `hono` to `hono/quick`:
- Complex regex patterns may not work
- Slight runtime performance decrease for complex routes

#### From `hono` to `hono/tiny`:  
- Regex constraints not supported
- Runtime performance decrease for complex patterns
- Significant reduction in bundle size

#### From `hono/quick` or `hono/tiny` to `hono`:
- Increased initialization time
- Better runtime performance for complex routes
- Support for all routing features

## Bundle Size Impact

**Approximate bundle size differences:**
- `hono/tiny`: Smallest (baseline)
- `hono/quick`: ~15-25% larger than tiny
- `hono`: ~30-50% larger than tiny

**Note**: Actual sizes depend on bundler and tree-shaking capabilities.

## See Also

- [Hono Application Specification](/docs/api/hono) - Core application API
- [Routing Specification](/docs/api/routing) - Route pattern syntax
- [Router Preset Examples](/docs/api/presets-examples) - Practical usage examples
- [Performance Benchmarks](/docs/concepts/benchmarks) - Detailed performance comparisons
