# Hono API Specification

Hono is a small, simple, and ultrafast web framework built on Web Standards. This specification documents the complete public API surface of Hono.

## Core Components

The Hono API consists of several key components:

- [Hono Object](/docs/api/hono) — The main application object that serves as the primary entry point
- [Routing System](/docs/api/routing) — The pattern-matching and request dispatching system
- [Context](/docs/api/context) — The per-request context with utilities for handling the current request
- [Request](/docs/api/request) — The enhanced request object (HonoRequest)
- [Exception Handling](/docs/api/exception) — Error handling and HTTP exceptions
- [Router Presets](/docs/api/presets) — Pre-configured router implementations for different environments

## What this spec provides

- Complete method/property signatures with types
- Parameters and their constraints
- Return values and types
- Caveats and edge cases
- Cross-links to related functionality

This specification is intended as a definitive reference for working with Hono.

## TypeScript support

Hono is built with TypeScript and provides extensive type support. Throughout this specification, you'll find type information for all APIs, enabling strong type safety and editor tooling.

## Examples

For practical, copy-paste patterns and walkthroughs, see:

- [Context Examples](/docs/api/context-examples)
- The general [Examples](/docs/examples/) section
