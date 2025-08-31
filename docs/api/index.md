# API Specification

This section contains the complete formal specification for Hono's API. Each component is documented with precise type definitions, method signatures, parameters, return values, and behavioral guarantees.

## Design Principles

Hono's API is built on Web Standards and follows these principles:
- **Standard-based**: Uses native Web API objects like `Request` and `Response`
- **Type-safe**: Full TypeScript support with proper generics
- **Minimal**: Small, focused API surface
- **Extensible**: Middleware and helper ecosystem

## Core API Components

### Application and Routing
- [**Hono**](/docs/api/hono) - Main application class with complete method signatures and constructor options
- [**Routing**](/docs/api/routing) - Comprehensive routing specification including path patterns, matching rules, and priority behavior

### Request and Response
- [**Context**](/docs/api/context) - Request-response lifecycle object with all methods and properties
- [**HonoRequest**](/docs/api/request) - Enhanced request object specification with all parsing methods

### Error Handling and Configuration  
- [**Exception**](/docs/api/exception) - HTTPException class specification and error handling patterns
- [**Presets**](/docs/api/presets) - Router preset specifications with performance characteristics

## Example Documentation

For practical usage examples and patterns, see the companion example files:
- [Context Examples](/docs/api/context-examples) - Practical Context usage patterns
- [Hono Examples](/docs/api/hono-examples) - Application configuration and integration examples
- [Routing Examples](/docs/api/routing-examples) - Advanced routing scenarios and patterns
- [Request Examples](/docs/api/request-examples) - Request parsing and validation examples
- [Exception Examples](/docs/api/exception-examples) - Error handling patterns
- [Preset Examples](/docs/api/presets-examples) - Router preset usage and comparisons

## Type Definitions

All API components support TypeScript generics for type safety:

```ts
type HonoEnv = {
  Bindings: { /* Environment bindings */ }
  Variables: { /* Per-request variables */ }
}
```

See individual component specifications for complete type information.
