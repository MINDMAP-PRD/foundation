# TypeScript / JavaScript / React / Node Standards

Use this reference when implementation work needs concrete coding conventions.

## 1. Code Quality Principles

### Readability First

- Code is read more than written.
- Prefer clear variable and function names.
- Prefer self-documenting code over commentary that restates the obvious.
- Keep formatting and structure consistent.

### KISS

- Prefer the simplest solution that clearly works.
- Avoid over-engineering and premature optimization.
- Easy to understand is better than clever.

### DRY

- Extract repeated logic into reusable functions or components.
- Share utilities where that reduces duplication without obscuring intent.
- Avoid copy-paste implementation drift across modules.

### YAGNI

- Do not build speculative features before they are needed.
- Start with the smallest sufficient implementation and refactor later if necessary.

## 2. TypeScript / JavaScript Standards

### Variable Naming

- Use descriptive names like `marketSearchQuery`, `isUserAuthenticated`, and `totalRevenue`.
- Avoid vague names like `x`, `flag`, `data1`, or `tmp` unless the scope is extremely local and obvious.

### Function Naming

- Prefer verb-noun names like `fetchMarketData`, `calculateSimilarity`, and `isValidEmail`.
- Avoid noun-only or vague names that hide intent.

### Immutability

- Prefer immutable updates with spread syntax or non-mutating array helpers.
- Do not mutate objects or arrays directly unless there is a documented performance reason.

### Error Handling

- Wrap async boundaries in `try/catch` when failure is expected or user-visible.
- Check response status before assuming success.
- Surface consistent, understandable error messages.
- Avoid swallowing errors silently.

### Async / Await

- Prefer `Promise.all` for independent async work that can safely run in parallel.
- Avoid serial awaits when they are not required by data dependency.

### Type Safety

- Prefer explicit interfaces, type aliases, discriminated unions, and precise return types.
- Avoid `any`; use `unknown` plus narrowing when necessary.
- Model status or lifecycle values with unions rather than loose strings when possible.

## 3. React Standards

### Components

- Use typed functional components.
- Keep props small, named clearly, and default only where that improves ergonomics.
- Prefer composition over deeply configurable monolith components.

### Hooks

- Extract reusable stateful behavior into custom hooks.
- Name hooks with the `use` prefix.
- Keep hook APIs focused and predictable.

### State Updates

- Use functional state updates when the next state depends on the previous one.
- Avoid stale closures and direct mutation.

### Rendering

- Prefer simple conditional rendering over nested ternary chains.
- Keep loading, error, empty, and success states explicit.

### UI Baseline

- For foundation-based UI work, use the repo's `shadcn/ui`-first baseline.
- Use custom components only when there is no suitable `shadcn/ui` primitive or composition path.

## 4. API / Backend Standards

### Route Conventions

- Use clear REST-style routes when the project/package expects HTTP APIs.
- Keep naming consistent across list, get, create, update, and delete operations.

### Response Shape

- Prefer one consistent response envelope per project or package.
- For example, a common shape may include `success`, `data`, `error`, and optional `meta`.
- Do not mix unrelated response formats without a documented reason.

### Input Validation

- Validate untrusted input at API boundaries.
- Prefer schema validation libraries such as `zod` where appropriate.
- Return consistent validation errors instead of ad hoc failures.

### Database Queries

- Select only the columns needed.
- Avoid `select('*')` unless there is a concrete reason and the contract expects it.

## 5. File Organization

Use the active project structure first. When the project/package does not specify one, prefer a clear separation like:

```text
src/
├── app/
├── components/
├── hooks/
├── lib/
├── types/
└── styles/
```

### Naming

- React components: `PascalCase.tsx`
- hooks: `camelCase` with `use` prefix
- utilities: `camelCase`
- type files: descriptive names such as `market.types.ts` when that pattern fits the project

## 6. Comments And Documentation

- Comment the reason or tradeoff, not the obvious action.
- Use JSDoc for public APIs or reusable exported functions when the contract benefits from explicit parameters, return values, exceptions, or examples.

## 7. Performance Guidance

- Memoize expensive computations or stable callbacks only when there is an actual benefit.
- Lazy-load heavy components or routes when that fits the product experience.
- Do not introduce memoization everywhere by reflex.

## 8. Testing Standards

### Test Shape

- Prefer Arrange / Act / Assert structure.
- Use descriptive test names that say what should happen and under what condition.

### What To Test

- important business logic
- validation boundaries
- error handling and fallback behavior
- security-sensitive flows when applicable
- critical UI state transitions

## 9. Code Smells To Watch For

### Long Functions

- Prefer smaller functions with focused responsibilities.

### Deep Nesting

- Prefer early returns and guard clauses.

### Magic Numbers

- Replace unexplained numeric values with named constants.

### Cleverness Drift

- If a solution is hard to explain, simplify it.

## 10. Review Checklist

Use this checklist during review or implementation:

- names are descriptive
- types are precise
- external input is validated
- async work is sequenced intentionally
- state updates avoid mutation
- React states are explicit
- APIs return a consistent shape
- files are organized consistently
- tests cover critical behavior
- complexity is proportional to the task

## 11. Foundation Integration Reminder

These standards improve implementation quality.

They do not override:

- foundation ownership boundaries
- governed package decisions
- module manifest rules
- security-specific requirements

If code quality guidance conflicts with the governed package or foundation contracts, resolve the ownership/contract issue first.
