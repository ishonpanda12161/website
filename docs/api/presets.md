# Router Presets Specification

Hono provides pre-configured router implementations optimized for different deployment environments and performance characteristics. Each preset uses the same Hono API but with different underlying routing engines.

## Available Presets

### `hono` (Default)

The default preset optimized for most production use cases.

```ts
import { Hono } from 'hono'
```

**Router implementation:**
```ts
new SmartRouter({
  routers: [new RegExpRouter(), new TrieRouter()]
})
```

**Characteristics:**
- Higher registration cost, excellent runtime performance
- Suitable for long-running server instances
- Best choice for most applications

### `hono/quick`

Fast startup preset for request-per-instance environments.

```ts
import { Hono } from 'hono/quick'
```

**Router implementation:**
```ts
new SmartRouter({
  routers: [new LinearRouter(), new TrieRouter()]
})
```

**Characteristics:**
- Fast registration, optimized for quick initialization
- Ideal when the app is bootstrapped per request
- Lower memory footprint during startup

### `hono/tiny`

Minimal preset for resource-constrained environments.

```ts
import { Hono } from 'hono/tiny'
```

**Router implementation:**
```ts
new PatternRouter()
```

**Characteristics:**
- Smallest bundle size
- Minimal memory usage
- Simplified routing algorithm

## Selection Guide

Choose the appropriate preset based on your deployment environment and performance requirements:

| Preset | Best For | Runtime Examples |
|--------|----------|------------------|
| `hono` | **Long-running servers** where startup cost is amortized over many requests | Deno, Bun, Node.js servers; Cloudflare Workers; Deno Deploy |
| `hono/quick` | **Per-request initialization** where fast startup is critical | Fastly Compute; AWS Lambda (cold starts) |
| `hono/tiny` | **Resource-constrained** environments prioritizing minimal bundle size | Edge computing; embedded systems; size-sensitive deployments |

## Performance Trade-offs

**Registration Performance:**
- `hono/quick` > `hono/tiny` > `hono`

**Runtime Performance:**
- `hono` > `hono/quick` > `hono/tiny`

**Bundle Size:**
- `hono/tiny` < `hono/quick` < `hono`

**Memory Usage:**
- `hono/tiny` < `hono/quick` < `hono`

## Migration Between Presets

All presets provide identical APIs. Migration requires only changing the import statement:

```ts
// From default preset
import { Hono } from 'hono'

// To quick preset  
import { Hono } from 'hono/quick'

// To tiny preset
import { Hono } from 'hono/tiny'
```

No code changes are required beyond the import statement.

## Custom Router Configuration

For advanced use cases, specify a custom router in the constructor:

```ts
import { Hono } from 'hono'
import { RegExpRouter } from 'hono/router/reg-exp-router'

const app = new Hono({ router: new RegExpRouter() })
```

## See Also

- [Hono Object](/docs/api/hono) — Constructor options and router configuration
- [Routing](/docs/api/routing) — Route patterns and matching behavior
