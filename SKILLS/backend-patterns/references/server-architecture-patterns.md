# Server Architecture Patterns

Use this reference when backend work needs concrete architecture guidance.

## 1. Layering

Choose the lightest architecture that fits the problem.

Common layers:

- transport or route handler
- middleware
- service or use-case layer
- repository or data-access layer

Do not add every layer automatically. Use them when they reduce coupling and clarify responsibilities.

## 2. Repository Pattern

Use a repository when:

- multiple callers need consistent data-access behavior
- query logic is becoming repetitive
- storage implementation details should be isolated

Avoid a repository layer when:

- the app is small and direct data access is still clear
- the repository would only wrap trivial one-line calls with no real abstraction benefit

## 3. Service Layer

Use a service layer for:

- business logic
- orchestration across multiple repositories or external systems
- domain rules that should not live in route handlers

Keep transport-specific concerns out of the service when possible.

## 4. Middleware

Use middleware for cross-cutting concerns such as:

- authentication
- request logging
- rate limiting
- tracing or request IDs
- shared validation entry points

Do not bury business logic inside middleware.

## 5. Query Optimization

Prefer:

- selecting only needed columns
- batching related fetches
- preloading related data deliberately
- using indexes that match real query paths

Watch for:

- `select('*')` with broad payloads
- N+1 loops
- hidden per-row fetches
- repeated query logic that should be centralized

## 6. Transactions

Use transactions for multi-step state changes that must succeed or fail together.

Record:

- what must stay atomic
- where retry is safe
- how failure is surfaced

## 7. Caching

Only add caching with a clear reason.

Common patterns:

- cache-aside
- read-through wrapper
- short-lived response caching

Always define:

- cache key shape
- TTL
- invalidation trigger
- stale-data tolerance

## 8. Background Jobs And Queues

Use background processing when work is:

- slow
- retryable
- external-service heavy
- not required to complete before the request returns

Design jobs to be:

- idempotent where practical
- observable
- failure-aware
- retry-safe

## 9. Centralized Error Handling

Prefer:

- typed operational errors for expected failures
- one shared error-mapping path for HTTP responses
- explicit handling for validation failures, conflicts, and internal errors

Avoid:

- ad hoc error formatting in every route
- leaking internal exception details

## 10. Logging And Observability

Prefer structured logs with stable fields such as:

- timestamp
- level
- message
- request ID
- user ID when appropriate
- route or job name

Log enough to diagnose failures without dumping sensitive internals.

## 11. Integration Boundaries

Keep external integrations behind explicit boundaries:

- provider adapters
- client wrappers
- queue publishers
- cache clients

That keeps backend modules easier to test and change.

## 12. Review Checklist

Before shipping backend structure changes, verify:

- responsibilities are not collapsed into one god object
- business logic is not trapped in route handlers
- query paths are intentional and efficient
- caching has an invalidation story
- jobs are failure-aware
- error handling is consistent
- logging is structured and useful
- security-sensitive behavior is reviewed by the appropriate security skill
- API contract shape is aligned with `api-design`
