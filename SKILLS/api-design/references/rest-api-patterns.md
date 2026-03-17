# REST API Patterns

Use this reference when API design work needs concrete conventions.

## 1. Resource Design

### URL Structure

Prefer:

```text
GET    /api/v1/users
GET    /api/v1/users/:id
POST   /api/v1/users
PUT    /api/v1/users/:id
PATCH  /api/v1/users/:id
DELETE /api/v1/users/:id
```

Use sub-resources for relationships:

```text
GET  /api/v1/users/:id/orders
POST /api/v1/users/:id/orders
```

Use action-style verbs sparingly for non-CRUD transitions:

```text
POST /api/v1/orders/:id/cancel
POST /api/v1/auth/login
POST /api/v1/auth/refresh
```

### Naming Rules

- resources should be plural
- URLs should be lowercase
- use kebab-case for multi-word resources
- avoid verbs in URLs unless the endpoint is truly an action and not a resource

Prefer:

- `/api/v1/team-members`
- `/api/v1/orders?status=active`
- `/api/v1/users/123/orders`

Avoid:

- `/api/v1/getUsers`
- `/api/v1/user`
- `/api/v1/team_members`
- `/api/v1/users/123/getOrders`

## 2. HTTP Methods And Status Codes

### Method Semantics

| Method | Idempotent | Safe | Use For |
|---|---|---|---|
| GET | Yes | Yes | Retrieve resources |
| POST | No | No | Create resources or trigger actions |
| PUT | Yes | No | Full replacement |
| PATCH | Usually no | No | Partial update |
| DELETE | Yes | No | Remove a resource |

### Status Codes

Success:

- `200 OK`
- `201 Created`
- `204 No Content`

Client errors:

- `400 Bad Request`
- `401 Unauthorized`
- `403 Forbidden`
- `404 Not Found`
- `409 Conflict`
- `422 Unprocessable Entity`
- `429 Too Many Requests`

Server errors:

- `500 Internal Server Error`
- `502 Bad Gateway`
- `503 Service Unavailable`

Do not flatten everything into `200`.

## 3. Response Format

### Success Response

```json
{
  "data": {
    "id": "abc-123",
    "email": "alice@example.com",
    "name": "Alice"
  }
}
```

### Collection Response

```json
{
  "data": [],
  "meta": {
    "total": 142,
    "page": 1,
    "per_page": 20,
    "total_pages": 8
  },
  "links": {
    "self": "/api/v1/users?page=1&per_page=20",
    "next": "/api/v1/users?page=2&per_page=20"
  }
}
```

### Error Response

```json
{
  "error": {
    "code": "validation_error",
    "message": "Request validation failed",
    "details": [
      {
        "field": "email",
        "message": "Must be a valid email address",
        "code": "invalid_format"
      }
    ]
  }
}
```

Prefer one response-envelope strategy per project/package.

## 4. Pagination

### Offset-Based

Use for:

- admin dashboards
- small datasets
- interfaces where users expect page numbers

### Cursor-Based

Use for:

- large datasets
- feeds
- infinite scroll
- public APIs that need stable scaling

Record which pagination mode is chosen and why.

## 5. Filtering, Sorting, And Search

Prefer explicit query conventions:

- equality filters: `?status=active`
- comparison filters: `?price[gte]=10&price[lte]=100`
- multiple values: `?category=electronics,clothing`
- sorting: `?sort=-created_at`
- search: `?q=wireless+headphones`
- field selection: `?fields=id,name,email`

Do not let each endpoint invent a different query style.

## 6. Authentication And Authorization

Authentication and authorization must be explicit per endpoint.

Typical patterns:

- bearer token in `Authorization` header
- API keys for service-to-service traffic
- ownership checks for resource-level access
- role/permission checks for privileged endpoints

This skill defines contract expectations only. Use security specialists for hardening.

## 7. Rate Limiting

If the API is exposed beyond tightly controlled internal callers, define:

- rate-limit tier
- rate-limit window
- rate-limit headers
- retry behavior

Common headers:

- `X-RateLimit-Limit`
- `X-RateLimit-Remaining`
- `X-RateLimit-Reset`
- `Retry-After`

## 8. Versioning

Prefer path versioning when versioning is needed:

```text
/api/v1/users
/api/v2/users
```

Guidance:

- start at `/api/v1/`
- do not create a new version for non-breaking additive changes
- use a new version for breaking response, field, URL, or auth changes
- keep deprecation policy explicit for public APIs

## 9. Input Validation

Validate request boundaries explicitly.

Prefer schema validation where appropriate and return structured validation errors instead of ambiguous failures.

## 10. Review Checklist

Before shipping a new endpoint, verify:

- URL follows naming rules
- HTTP method is correct
- status codes are semantic
- input is validated
- error response shape is consistent
- list endpoints paginate deliberately
- auth and authorization are explicit
- rate limiting is addressed when needed
- internal details are not leaked
- versioning is aligned with the project/package strategy
