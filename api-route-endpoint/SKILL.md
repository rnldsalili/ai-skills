---
name: api-route-endpoint
description: Create or update Hono API routes/endpoints in apps/api for this repo (Cloudflare Workers). Use when adding/editing route index files, handlers, or Zod schemas, wiring middleware, Prisma access, validation, and the standard JSON response shape.
---

# API Route/Endpoint Skill

Follow existing Hono patterns in apps/api when creating or updating endpoints.

## Workflow

- Identify the feature folder under `apps/api/src/routes` and create or update the three files:
  - `routes/<feature>/<feature>.index.ts` (route wiring)
  - `routes/<feature>/<feature>.handler.ts` (handlers)
  - `routes/<feature>/<feature>.schema.ts` (Zod schemas)
- Use `createRouter()` for route modules and `createHandlers()` for handlers.
- Use `validate()` from `apps/api/src/lib/validator.ts` with Zod schemas.
- Use `initializePrisma(c.env.DATABASE_URL)` for database access.
- Return JSON with the standard response structure (see references).
- Wire middleware for auth/roles where needed.
- Register the route in `apps/api/src/routes/index.ts` if it is a new top-level group.

## Required patterns

- Route wiring: use `.get`, `.post`, `.put`, `.delete` with spread handler arrays: `route.get('/', ...getItems)`.
- Validation: `validate('param' | 'query' | 'json', schema)` as the first handlers in `createHandlers`.
- Handler response: return `c.json({ meta: { code, message }, data: { ... } }, code)`.
- Errors: map Prisma errors to status codes where appropriate; fall through to app error handler.

## Middleware

- Use `requireAuth` for authenticated routes.
- Use `requireAdminRole` for admin-only routes.

## Response structure

- Always return `{ meta: { code, message }, data: { ... } }`.
- For errors with no payload, return `data: {}`.

## References

- See `references/api-route-patterns.md` for concrete file layouts, schema patterns, and response examples.
