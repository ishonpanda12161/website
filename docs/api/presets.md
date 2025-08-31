# Presets Specification

Hono provides preconfigured presets that optimize router selection for different use cases and runtime environments.

## Overview

Presets are Hono packages with pre-selected router combinations optimized for specific scenarios. Each preset exports the same `Hono` class with identical APIs, differing only in the underlying router implementation.

**Interoperability:** All presets are fully interchangeable and use the same API surface.

## Available Presets

### `hono` (Default)

The standard Hono preset with optimal performance for most scenarios.

**Import:**

```ts
import { Hono } from 'hono'
```

**Router Configuration:**

```ts
new SmartRouter({
  routers: [new RegExpRouter(), new TrieRouter()],
})
```

**Characteristics:**

- **Registration Phase:** Moderate performance
- **Runtime Performance:** Excellent performance after initialization
- **Memory Usage:** Moderate memory footprint
- **Optimization:** Automatic router selection based on route patterns

**Best For:**

- Long-running servers (Node.js, Bun, Deno)
- Cloudflare Workers with persistent isolates
- Applications with complex routing patterns
- Production deployments requiring optimal runtime performance

### `hono/quick`

Optimized for fast initialization with good runtime performance.

**Import:**

```ts
import { Hono } from 'hono/quick'
```

**Router Configuration:**

```ts
new SmartRouter({
  routers: [new LinearRouter(), new TrieRouter()],
})
```

**Characteristics:**

- **Registration Phase:** Fast initialization
- **Runtime Performance:** Good performance
- **Memory Usage:** Lower memory overhead
- **Optimization:** Linear search for simple patterns, trie for complex ones

**Best For:**

- Environments with per-request initialization
- Fastly Compute applications
- Serverless functions with cold starts
- Development environments

### `hono/tiny`

Minimal preset with the smallest bundle size.

**Import:**

```ts
import { Hono } from 'hono/tiny'
```

**Router Configuration:**

```ts
new PatternRouter()
```

**Characteristics:**

- **Registration Phase:** Minimal overhead
- **Runtime Performance:** Basic but adequate
- **Memory Usage:** Lowest memory footprint
- **Optimization:** Simple pattern matching

**Best For:**

- Resource-constrained environments
- Applications with minimal routing needs
- Bundle size optimization requirements
- Embedded or edge computing scenarios

## Router Implementations

### SmartRouter

Automatically selects the optimal router based on route patterns and count.

**Features:**

- Adaptive router selection
- Combines multiple router strategies
- Optimizes based on route complexity
- Balances initialization and runtime performance

### RegExpRouter

Uses regular expressions for route matching.

**Features:**

- Efficient pattern matching
- Good performance for complex routes
- Handles regex patterns in routes
- Moderate memory usage

### TrieRouter

Uses a trie data structure for efficient route lookup.

**Features:**

- Fast lookup for large route sets
- Memory efficient storage
- Excellent for prefix matching
- Scales well with route count

### LinearRouter

Simple linear search through routes.

**Features:**

- Fast initialization
- Low memory overhead
- Simple implementation
- Good for small route sets

### PatternRouter

Basic pattern matching with minimal overhead.

**Features:**

- Minimal code footprint
- Simple string matching
- Lowest resource usage
- Limited feature set

## Selection Guide

### Performance Characteristics

| Preset       | Initialization | Runtime   | Memory   | Bundle Size |
| ------------ | -------------- | --------- | -------- | ----------- |
| `hono`       | Moderate       | Excellent | Moderate | Standard    |
| `hono/quick` | Fast           | Good      | Low      | Standard    |
| `hono/tiny`  | Minimal        | Basic     | Minimal  | Smallest    |

### Environment Recommendations

#### Long-Life Servers

**Recommended:** `hono` (default)

**Environments:**

- Node.js with Express-like usage
- Bun HTTP servers
- Deno HTTP servers
- Long-running processes

**Reasoning:** The initialization cost is amortized over many requests, making the excellent runtime performance worthwhile.

#### Per-Request Initialization

**Recommended:** `hono/quick`

**Environments:**

- Fastly Compute
- Some serverless platforms
- Request-scoped applications

**Reasoning:** Fast initialization is critical when the application is recreated for each request.

#### Edge/Isolated Environments

**Recommended:** `hono` (default)

**Environments:**

- Cloudflare Workers
- Deno Deploy
- Vercel Edge Runtime

**Reasoning:** These environments typically reuse isolates/instances, making runtime performance more important than initialization speed.

#### Resource-Constrained Environments

**Recommended:** `hono/tiny`

**Environments:**

- Embedded systems
- IoT devices
- Memory-limited containers
- Bundle-size sensitive applications

**Reasoning:** Minimal resource usage is the primary concern.

## Migration Between Presets

### Import Changes Only

Switching between presets requires only changing the import statement:

```ts
// From default to quick
import { Hono } from 'hono' // Before
import { Hono } from 'hono/quick' // After

// From quick to tiny
import { Hono } from 'hono/quick' // Before
import { Hono } from 'hono/tiny' // After
```

### API Compatibility

All presets maintain 100% API compatibility:

- Same method signatures
- Same TypeScript types
- Same middleware compatibility
- Same feature set (except router-specific optimizations)

### Performance Testing

When switching presets, benchmark your specific use case:

```ts
// Performance testing template
const iterations = 10000
const start = Date.now()

for (let i = 0; i < iterations; i++) {
  // Your routing operations
}

const duration = Date.now() - start
console.log(`${iterations} operations took ${duration}ms`)
```

## Custom Router Configuration

### Manual Router Selection

Override the default router for any preset:

```ts
import { Hono } from 'hono'
import { LinearRouter } from 'hono/router/linear-router'

const app = new Hono({
  router: new LinearRouter(),
})
```

### Router Comparison

Test different routers with your specific route patterns:

```ts
import { RegExpRouter } from 'hono/router/reg-exp-router'
import { TrieRouter } from 'hono/router/trie-router'
import { LinearRouter } from 'hono/router/linear-router'

const routers = [
  new RegExpRouter(),
  new TrieRouter(),
  new LinearRouter(),
]

// Benchmark each router with your routes
```

## Bundle Analysis

### Size Comparison

| Preset       | Additional Bundle Size | Router Complexity |
| ------------ | ---------------------- | ----------------- |
| `hono`       | Baseline               | High              |
| `hono/quick` | Similar                | Medium            |
| `hono/tiny`  | Smallest (-30-50%)     | Low               |

### Tree Shaking

All presets support proper tree shaking:

- Unused routers are eliminated
- Only imported functionality is bundled
- Middleware can be selectively imported

## Runtime Characteristics

### Memory Usage Patterns

**`hono`:** Higher initial memory usage, stable during runtime
**`hono/quick`:** Moderate memory usage, efficient for frequent initialization
**`hono/tiny`:** Minimal memory footprint throughout lifecycle

### CPU Performance

**Route Registration:**

- `hono/quick` > `hono/tiny` > `hono`

**Route Matching:**

- `hono` > `hono/quick` > `hono/tiny`

**Overall Throughput:**

- For high-traffic: `hono` > `hono/quick` > `hono/tiny`
- For low-traffic: Differences are negligible

## Best Practices

### Preset Selection Strategy

1. **Start with defaults** - Use `hono` unless you have specific constraints
2. **Measure actual performance** - Benchmark with your real application
3. **Consider deployment environment** - Match preset to runtime characteristics
4. **Monitor bundle size** - Use `hono/tiny` if size is critical

### Development vs Production

**Development:** Any preset works well; choose based on hot reload speed
**Production:** Choose based on deployment environment and performance requirements

### Multi-Environment Applications

Consider using different presets for different deployment targets:

```ts
// package.json scripts
{
  "build:edge": "build with hono/tiny",
  "build:server": "build with hono",
  "build:serverless": "build with hono/quick"
}
```

## See Also

- [Hono Specification](/docs/api/hono) - Router configuration options
- [Routing Specification](/docs/api/routing) - Routing patterns and performance
- [Benchmarks](/docs/concepts/benchmarks) - Performance comparisons
