# API Reference

Complete API specification for Hono - a small, simple, and ultrafast web framework built on Web Standards.

## Overview

Hono's API is built on Web Standards, making it familiar and easy to understand. This reference provides complete specifications for all Hono APIs, including method signatures, parameters, return values, and behavioral details.

## Core APIs

### [Hono Application](/docs/api/hono)
The main `Hono` class for creating applications, handling routing, middleware, and request processing.

**Key Features:**
- Application lifecycle management
- Route registration and handling
- Middleware integration
- Error handling and not found responses
- Environment and variable management

### [Context Object](/docs/api/context)
The `Context` object provided to handlers and middleware, containing request/response helpers and application state.

**Key Features:**
- Request and response manipulation
- Header and status code management  
- Body parsing and response generation
- Per-request variables and environment access
- Rendering and platform-specific features

### [HonoRequest](/docs/api/request)
Enhanced request object that wraps the standard `Request` with additional parsing and access methods.

**Key Features:**
- Path and query parameter access
- Header parsing and validation
- Body parsing (JSON, form data, text, etc.)
- Route matching information
- Raw request access

### [Routing System](/docs/api/routing)
Flexible routing system supporting various HTTP methods, path patterns, and route organization.

**Key Features:**
- HTTP method routing (GET, POST, etc.)
- Path parameters and wildcards
- Optional parameters and regex patterns
- Route grouping and base paths
- Priority and ordering rules

### [Exception Handling](/docs/api/exception)
HTTP exception system for structured error handling and custom error responses.

**Key Features:**
- `HTTPException` class for HTTP errors
- Custom error messages and responses
- Error cause tracking
- Integration with error handlers

### [Router Presets](/docs/api/presets)
Pre-configured router combinations optimized for different deployment scenarios.

**Key Features:**
- `hono` - Default smart router for most use cases
- `hono/quick` - Fast initialization for per-request environments
- `hono/tiny` - Minimal router for resource-constrained environments

## Usage Examples

Each API specification includes links to comprehensive examples:
- [Hono Examples](/docs/api/hono-examples) - Application setup and configuration
- [Context Examples](/docs/api/context-examples) - Request/response handling patterns
- [Request Examples](/docs/api/request-examples) - Request parsing and validation
- [Routing Examples](/docs/api/routing-examples) - Route patterns and organization
- [Exception Examples](/docs/api/exception-examples) - Error handling strategies
- [Preset Examples](/docs/api/presets-examples) - Router configuration for different platforms

## TypeScript Support

All APIs are fully typed with TypeScript, including:
- Generic type parameters for environment bindings and variables
- Accurate return types for all methods
- Type-safe parameter access and validation
- IntelliSense support in compatible editors

## Web Standards Compatibility

Hono is built on Web Standards and is compatible with:
- [Request](https://developer.mozilla.org/en-US/docs/Web/API/Request) and [Response](https://developer.mozilla.org/en-US/docs/Web/API/Response) objects
- [Headers](https://developer.mozilla.org/en-US/docs/Web/API/Headers) interface
- [URL](https://developer.mozilla.org/en-US/docs/Web/API/URL) and [URLSearchParams](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams)
- Standard HTTP methods and status codes
