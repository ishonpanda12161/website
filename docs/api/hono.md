# App — Hono Specification

The `Hono` class is the primary entry point for creating a Hono application. It handles routing, middleware registration, error handling, and serves as the application container.

## Constructor

```ts
new Hono<Env, BasePath>(options?)
```

Parameters:
- options (optional): object
  - strict: boolean — When true (default), `/path` and `/path/` are treated as different routes.
  - router: Router — Custom router implementation (see [Router Presets](/docs/api/presets)).
  - getPath: (req: Request) => string — Custom function to extract a path from the request.

Returns: Hono

Type parameters:
- Env — Environment bindings type (for platform bindings like Cloudflare Workers).
- BasePath — Base path type parameter for type-safe routing.

## Routing methods

All route registration methods return the app instance for chaining.

```ts
app.get(path, ...handlers)
app.post(path, ...handlers)
app.put(path, ...handlers)
app.delete(path, ...handlers)
app.patch(path, ...handlers)
app.options(path, ...handlers)
app.head(path, ...handlers)
```

- path: string | string[] — URL path pattern(s) to match.
- handlers: ...MiddlewareHandler — One or more handlers/middleware.

### Generic routing

```ts
app.on(method, path, ...handlers)
```
- method: string | string[] — HTTP method(s) to match (e.g., 'GET', ['PUT','PATCH']).
- path: string | string[] — Path pattern(s).
- handlers: ...MiddlewareHandler

```ts
app.all(path, ...handlers)
```
- Registers handlers for all HTTP methods for the given path(s).

## Middleware

```ts
app.use([path], ...middleware)
```
- path (optional): string | string[] — Path pattern(s) to apply middleware to.
- middleware: ...MiddlewareHandler

Middleware executes before route handlers and in registration order.

## Grouping & composition

```ts
app.route(path, subApp)
```
- Mount another Hono app at the given path prefix.

```ts
app.basePath(path)
```
- Set a base path for all routes in the app.

## Special handlers

```ts
app.notFound(handler)
```
- handler: (c: Context) => Response | Promise<Response> — Custom 404 response handler.

```ts
app.onError(handler)
```
- handler: (err: Error, c: Context) => Response | Promise<Response> — Global error handler.

## Integration

```ts
app.mount(path, handler)
```
- Mount another web framework or generic handler under a prefix.

## Execution

```ts
app.fetch(request, env?, event?)
```
- request: Request — Incoming request.
- env (optional): object — Environment bindings.
- event (optional): object — Execution/fetch event context.

Returns: Promise<Response>

```ts
app.request(pathOrRequest, options?, env?)
```
- For testing: send a request (string path or Request object) to the app.

Returns: Promise<Response>

```ts
app.fire()
```
Deprecated — Use `fire()` from `hono/service-worker` instead. Automatically adds a global `fetch` event listener for Service Worker environments.

## Strict mode

Strict mode defaults to true and distinguishes `/hello` from `/hello/`.
Set `strict: false` in the constructor options to treat both as equal.

## Router option

Specify a router explicitly by passing the `router` option (see [Router Presets](/docs/api/presets)).

## Generics

Pass generics to specify types for platform bindings and per-request variables used with `c.set`/`c.get`.

```ts
const app = new Hono<{ Bindings: MyBindings; Variables: MyVariables }>()
```

Related: [Routing](/docs/api/routing), [Context](/docs/api/context), [Exception Handling](/docs/api/exception), [Router Presets](/docs/api/presets)
