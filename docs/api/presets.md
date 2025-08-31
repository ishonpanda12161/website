# Router Presets Specification

Router presets provide pre-configured Hono instances optimized for different deployment environments and use cases. Each preset uses a different combination of routing algorithms for optimal performance.

## Overview

Hono provides three main presets that differ only in their routing strategy:
- **Default (`hono`)**: Optimized for long-running servers
- **Quick (`hono/quick`)**: Optimized for per-request initialization  
- **Tiny (`hono/tiny`)**: Optimized for minimal bundle size

All presets export the same `Hono` class with identical API - only the internal router differs.

## Default Preset (`hono`)

### Import

```ts
import { Hono } from 'hono'
```

### Router Configuration

```ts
new SmartRouter({
  routers: [new RegExpRouter(), new TrieRouter()]
})
```

### Characteristics

- **Route Registration**: Slower (builds optimized data structures)
- **Route Resolution**: Fastest for most patterns
- **Memory Usage**: Higher (pre-computed structures)
- **Bundle Size**: ~35KB minified
- **Supported Patterns**: All patterns including complex regex

### Performance Profile

- **Optimization Phase**: Routes are compiled into efficient lookup structures
- **Runtime Performance**: O(1) for simple patterns, O(log n) for complex patterns
- **Best Case**: Applications with many routes and high request volume
- **Worst Case**: Applications with frequent route registration changes

### Suitable Platforms

- **Node.js** servers (Express-style applications)
- **Deno** applications with persistent processes
- **Bun** HTTP servers
- **Cloudflare Workers** (isolates persist across requests)
- **Deno Deploy** (similar isolate behavior)
- **Vercel Edge Functions** (with isolate persistence)

### Use Cases

- Production web servers
- APIs with complex routing patterns
- Applications with high request volume
- Long-running server processes
- Applications using advanced routing features

## Quick Preset (`hono/quick`)

### Import

```ts
import { Hono } from 'hono/quick'
```

### Router Configuration

```ts
new SmartRouter({
  routers: [new LinearRouter(), new TrieRouter()]
})
```

### Characteristics

- **Route Registration**: Fast (minimal preprocessing)
- **Route Resolution**: Good performance for most use cases  
- **Memory Usage**: Moderate (less pre-computation)
- **Bundle Size**: ~30KB minified
- **Supported Patterns**: All patterns with good performance

### Performance Profile

- **Initialization Speed**: Optimized for quick startup
- **Runtime Performance**: O(n) for linear search, O(log n) for tree patterns
- **Best Case**: Applications initialized per request
- **Worst Case**: Applications with hundreds of routes

### Suitable Platforms

- **Fastly Compute@Edge** (per-request initialization)
- **AWS Lambda** (cold start optimization)
- **Google Cloud Functions** (function initialization)
- **Azure Functions** (startup performance critical)
- **Edge runtimes** with per-request lifecycle

### Use Cases

- Serverless functions
- Edge computing applications
- APIs with moderate complexity
- Applications where cold start time matters
- Microservices with focused routing

## Tiny Preset (`hono/tiny`)

### Import

```ts
import { Hono } from 'hono/tiny'
```

### Router Configuration

```ts
new PatternRouter()
```

### Characteristics

- **Route Registration**: Fastest (no preprocessing)
- **Route Resolution**: Linear search performance
- **Memory Usage**: Minimal (no pre-computed structures)
- **Bundle Size**: ~15KB minified
- **Supported Patterns**: Basic patterns (limited regex support)

### Performance Profile

- **Initialization Speed**: Fastest startup time
- **Runtime Performance**: O(n) linear search
- **Best Case**: Applications with few routes (<20)
- **Worst Case**: Applications with many routes (>50)

### Pattern Limitations

- Complex regex patterns may not be supported
- Performance degrades linearly with route count
- Best suited for simple parameter patterns

### Suitable Platforms

- **IoT devices** (memory-constrained)
- **Embedded systems** (resource limitations)
- **Cloudflare Workers** (bundle size limits)
- **Mobile applications** (size-sensitive)
- **Browser environments** (client-side routing)

### Use Cases

- Simple APIs or microservices
- Prototype applications
- Resource-constrained environments
- Bundle size-critical applications
- Client-side routing scenarios

## Router Algorithm Details

### SmartRouter

**Used in**: Default and Quick presets

**Behavior**: Automatically selects the best router from its collection based on registered routes.

**Selection Logic**:
1. Analyzes route patterns during registration
2. Chooses optimal router for the pattern set
3. Falls back to TrieRouter for unsupported patterns

### RegExpRouter

**Used in**: Default preset

**Characteristics**:
- Compiles all routes into a single regular expression
- O(1) lookup time for most patterns
- Highest memory usage during compilation
- Best performance for high route counts

**Limitations**:
- Not all patterns supported
- Complex compilation phase

### TrieRouter

**Used in**: Default and Quick presets

**Characteristics**:
- Uses trie (prefix tree) data structure
- O(log n) lookup time
- Supports all route patterns
- Balanced memory usage

**Behavior**:
- Tree traversal for route matching
- Fallback option for unsupported patterns in other routers

### LinearRouter

**Used in**: Quick preset

**Characteristics**:
- Simple sequential route matching
- O(n) lookup time
- Minimal memory overhead
- Fast initialization

**Trade-offs**:
- Performance degrades with route count
- Simple implementation
- No preprocessing required

### PatternRouter

**Used in**: Tiny preset

**Characteristics**:
- Simplest possible routing implementation
- O(n) sequential search
- Minimal bundle size impact
- Basic pattern support only

**Limitations**:
- Limited regex pattern support
- Linear performance degradation
- No optimization features

## Selection Guidelines

### Choose Default (`hono`) When:
- Building production web servers
- Application has >20 routes
- Using complex routing patterns
- High request volume expected  
- Long-running server processes
- Bundle size is not critical

### Choose Quick (`hono/quick`) When:
- Deploying to serverless platforms
- Cold start performance is important
- Application initialized per request
- Moderate routing complexity
- Edge computing deployment
- Fastly Compute@Edge platform

### Choose Tiny (`hono/tiny`) When:
- Bundle size is critical (<20KB total)
- Simple routing requirements (<20 routes)
- Resource-constrained environments
- IoT or embedded applications
- Prototype or demo applications
- Client-side routing needs

## Migration Considerations

### API Compatibility

All presets maintain 100% API compatibility:
```ts
// These are functionally equivalent
import { Hono } from 'hono'
import { Hono } from 'hono/quick'  
import { Hono } from 'hono/tiny'
```

### Pattern Support

When migrating to `hono/tiny`, some complex patterns may need simplification:

```ts
// Complex pattern (may not work with tiny)
app.get('/users/:id{[0-9]+}', handler)

// Simplified pattern (works with all presets)
app.get('/users/:id', handler)
```

### Performance Testing

Benchmark different presets with your specific route patterns:

```ts
const routes = [/* your routes */]
const testRequests = [/* your request patterns */]

// Test each preset with your workload
const presets = ['hono', 'hono/quick', 'hono/tiny']
for (const preset of presets) {
  const { Hono } = await import(preset)
  // Benchmark route registration and request handling
}
```

## Bundle Size Analysis

### Approximate Sizes (minified + gzipped)

- **hono**: ~35KB total (~12KB gzipped)
- **hono/quick**: ~30KB total (~10KB gzipped)
- **hono/tiny**: ~15KB total (~6KB gzipped)

### Dependencies

Each preset includes different router implementations, affecting total bundle size when tree-shaking is applied.

## Performance Benchmarks

### Route Registration (1000 routes)
- **Default**: ~50ms (with optimization)
- **Quick**: ~20ms (minimal processing)
- **Tiny**: ~5ms (no processing)

### Request Resolution (per request)
- **Default**: ~0.1ms (optimized lookup)
- **Quick**: ~0.3ms (smart selection)
- **Tiny**: ~2ms (linear search)

*Note: Actual performance varies by route complexity and request patterns*

## See Also

- [Router Preset Examples](/docs/api/presets-examples) - Practical usage patterns and comparisons
- [Hono Specification](/docs/api/hono) - Core application class specification
- [Routing Specification](/docs/api/routing) - Route pattern details and matching
- [Performance Guide](/docs/guides/best-practices) - Optimization strategies
