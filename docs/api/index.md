# API Reference

This section provides the complete API specification for Hono, a lightweight web framework built on Web Standards.

## Core Components

### [Hono](/docs/api/hono)
The main application class for creating HTTP servers and handling requests.

**Key Features:**
- HTTP method routing (GET, POST, PUT, DELETE, etc.)
- Middleware support
- Error handling
- Platform-agnostic design

### [Context](/docs/api/context)
The request-response context object available in all handlers and middleware.

**Key Features:**
- Request and response helpers
- Environment variable access
- Per-request variable storage
- Response generation utilities

### [HonoRequest](/docs/api/request)
Enhanced request object with convenient methods for accessing request data.

**Key Features:**
- Path parameter extraction
- Query string parsing
- Request body parsing
- Header access

### [Routing](/docs/api/routing)
Flexible routing system supporting various path patterns and HTTP methods.

**Key Features:**
- Path parameters and wildcards
- Regular expression patterns
- Route grouping and chaining
- Priority-based matching

### [Exception](/docs/api/exception)
HTTP exception handling for error responses and custom error processing.

**Key Features:**
- HTTPException class for structured errors
- Custom error responses
- Error cause tracking

### [Presets](/docs/api/presets)
Pre-configured Hono instances optimized for different runtime environments.

**Key Features:**
- Router selection for different use cases
- Performance optimization
- Platform-specific configurations

## Examples and Guides

For practical usage examples and implementation patterns, see:

- [Hono Examples](/docs/api/hono-examples) - Practical application patterns
- [Context Examples](/docs/api/context-examples) - Request handling examples  
- [Request Examples](/docs/api/request-examples) - Data parsing patterns
- [Routing Examples](/docs/api/routing-examples) - Advanced routing patterns

## Type Definitions

Hono is built with TypeScript and provides full type safety. All APIs include comprehensive type definitions for:

- Generic type parameters for environment bindings
- Context variable typing
- Request/response type safety
- Middleware type inference
