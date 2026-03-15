---
name: api-design
description: REST API design patterns, HTTP methods, status codes, pagination, and error responses. Auto-triggers when creating or modifying API routes, endpoints, or handlers.
compatibility: Designed for Claude Code
---

# API Design

Auto-activates when creating or modifying API routes, endpoints, or handlers.

## Resource Naming

- Plural nouns: `/api/students`, `/api/courses`
- Nested for relationships: `/api/courses/:id/enrollments`
- Query params for filtering: `/api/courses?department=cs&active=true`
- Never verbs in URLs: `/api/users` not `/api/getUsers`

## HTTP Methods

| Method | Purpose | Idempotent | Response |
|--------|---------|------------|----------|
| GET | Read resource(s) | Yes | 200 + data |
| POST | Create resource | No | 201 + created resource |
| PUT | Full replace | Yes | 200 + updated resource |
| PATCH | Partial update | Yes | 200 + updated resource |
| DELETE | Remove resource | Yes | 204 (no body) |

## Status Codes

HTTP status IS the success/failure indicator. No envelope needed.

```
2xx Success
  200 OK              — GET, PUT, PATCH (return data directly)
  201 Created         — POST (include Location header)
  204 No Content      — DELETE

4xx Client Error
  400 Bad Request     — Validation failed
  401 Unauthorized    — Missing or invalid auth
  403 Forbidden       — Authenticated but not authorized
  404 Not Found       — Resource doesn't exist
  409 Conflict        — Duplicate, state conflict
  422 Unprocessable   — Semantically invalid
  429 Too Many Reqs   — Rate limited (include Retry-After header)

5xx Server Error
  500 Internal Error  — Generic (never expose internals)
  503 Unavailable     — Maintenance or overload
```

## Response Format

**Success** — return data directly, status code tells the story:

```typescript
// GET /api/courses/cs101
return Response.json({ id: "cs101", name: "Intro to CS", credits: 3 })

// GET /api/courses?page=2&limit=20 (with pagination meta)
return Response.json({
  data: courses,
  meta: { total: 150, page: 2, limit: 20 }
})
```

**Error** — standard error struct with type and message:

```typescript
interface ApiError {
  error: string    // Machine-readable error type (e.g., "validation_error", "not_found")
  message: string  // Human-readable description
}

// 400
return Response.json(
  { error: "validation_error", message: "Email is required" },
  { status: 400 }
)

// 404
return Response.json(
  { error: "not_found", message: "Course not found" },
  { status: 404 }
)
```

## Pagination

Use cursor-based for large/real-time datasets, offset-based for simple cases:

```
# Offset-based
GET /api/items?page=2&limit=20

# Cursor-based
GET /api/items?cursor=abc123&limit=20
```

## Validation

Always validate request body at the route handler boundary:

```typescript
import { z } from 'zod'

const CreateCourseSchema = z.object({
  name: z.string().min(1).max(200),
  department: z.enum(['cs', 'math', 'eng']),
  credits: z.number().int().min(1).max(6),
})

const body = CreateCourseSchema.parse(await request.json())
```

## Rate Limiting

Include headers in responses:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1620000000
Retry-After: 30  (on 429)
```
