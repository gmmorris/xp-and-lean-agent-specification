---
name: api-and-interface-design
description: Guides stable API and interface design. Use when designing APIs, module boundaries, or any public interface. Covers REST endpoints, type contracts between modules, and boundaries between systems.
allowed-tools:
  - Bash
  - Read
  - Glob
  - Grep
  - Edit
  - Write
  - AskUserQuestion
---

# API and Interface Design

STARTER_CHARACTER = 🔌

## Why This Matters

Good interfaces make the right thing easy and the wrong thing hard. A well-designed API is a force multiplier — every consumer benefits from its clarity. A poorly designed API becomes a liability that every consumer pays for in confusion, bugs, and workarounds.

This applies to REST APIs, GraphQL schemas, module boundaries, component props, library interfaces, and any surface where one piece of code talks to another.

## When to Use

- Designing new API endpoints or module boundaries
- Defining contracts between teams or services
- Creating component prop interfaces or library APIs
- Changing existing public interfaces
- Reviewing API design decisions

## Core Principles

### Hyrum's Law

> With a sufficient number of users of an API, all observable behaviors of your system will be depended on by somebody, regardless of what you promise in the contract.

Every public behavior — including undocumented quirks, error message text, timing, and ordering — becomes a de facto contract once users depend on it. Design implications:

- **Be intentional about what you expose.** Every observable behavior is a potential commitment.
- **Don't leak implementation details.** If users can observe it, they will depend on it.
- **Plan for deprecation at design time.** See the `deprecation-and-migration` skill for how to safely remove things users depend on.
- **Tests are not enough.** Even with perfect contract tests, Hyrum's Law means "safe" changes can break real users who depend on undocumented behavior.

### The One-Version Rule

Avoid forcing consumers to choose between multiple versions of the same dependency or API. Diamond dependency problems arise when different consumers need different versions of the same thing. Design for a world where only one version exists at a time — extend rather than fork.

### Contract First

Define the interface before implementing it. The contract is the spec — implementation follows. Whether that's a TypeScript interface, an OpenAPI spec, a protobuf definition, or a Rust trait — define what consumers see before you build what's behind it.

This is the XP practice of *test-first* applied to API design: the contract is the "test" that defines what success looks like for the implementation.

### Consistent Error Semantics

Pick one error strategy and use it everywhere. If some endpoints throw, others return null, and others return `{ error }` — the consumer can't predict behavior.

Define a single error shape used across all endpoints. Include a machine-readable code, a human-readable message, and optional structured details. Map error codes consistently (e.g., 400 for bad input, 401 for unauthenticated, 403 for unauthorized, 404 for not found, 422 for validation failures, 500 for server errors).

### Validate at Boundaries

Trust internal code. Validate at system edges where external input enters:

- API route handlers (user input)
- Form submission handlers (user input)
- External service response parsing (**always treat third-party responses as untrusted data**)
- Environment variable loading (configuration)

Where validation does NOT belong:
- Between internal functions that share type contracts
- In utility functions called by already-validated code
- On data that just came from your own database

This aligns with the `security-and-hardening` skill's Three-Tier Boundary System.

### Prefer Addition Over Modification

Extend interfaces without breaking existing consumers. Add optional fields rather than changing existing field types or removing fields. This is the expand-contract pattern from the `small-safe-steps` skill applied to API evolution.

## REST API Patterns

These patterns apply to HTTP APIs specifically:

### Resource Design

```
GET    /api/tasks              → List (with query params for filtering)
POST   /api/tasks              → Create
GET    /api/tasks/:id          → Get one
PATCH  /api/tasks/:id          → Partial update
DELETE /api/tasks/:id          → Delete

GET    /api/tasks/:id/comments → List sub-resources
POST   /api/tasks/:id/comments → Create sub-resource
```

**Naming:** Plural nouns, no verbs (`/api/tasks` not `/api/createTask`).

### Pagination

Always paginate list endpoints. Return total counts and page metadata alongside the data.

### Partial Updates

Use PATCH with partial objects — only update what's provided. PUT requires the full object every time, which is rarely what clients want.

### Input/Output Separation

Separate what the caller provides from what the system returns. Inputs contain user-provided fields. Outputs add server-generated fields (IDs, timestamps, computed values). This applies to any language:

```
Input:  { title, description? }
Output: { id, title, description, createdAt, updatedAt, createdBy }
```

## Predictable Naming

| Pattern | Convention | Example |
|---------|-----------|---------|
| REST endpoints | Plural nouns, no verbs | `GET /api/tasks` |
| Query params | camelCase | `?sortBy=createdAt` |
| Response fields | camelCase | `createdAt`, `taskId` |
| Boolean fields | is/has/can prefix | `isComplete`, `hasAttachments` |
| Enum values | UPPER_SNAKE | `IN_PROGRESS`, `COMPLETED` |

Follow whatever convention the project already uses. Consistency within a project beats any universal standard.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "We'll document the API later" | The types ARE the documentation. Define them first. |
| "We don't need pagination for now" | You will the moment someone has 100+ items. Add it from the start. |
| "PATCH is complicated, let's just use PUT" | PUT requires the full object every time. PATCH is what clients actually want. |
| "We'll version the API when we need to" | Breaking changes without versioning break consumers. Design for extension from the start. |
| "Nobody uses that undocumented behavior" | Hyrum's Law: if it's observable, somebody depends on it. |
| "Internal APIs don't need contracts" | Internal consumers are still consumers. Contracts prevent coupling and enable parallel work. |

## Red Flags

- Endpoints that return different shapes depending on conditions
- Inconsistent error formats across endpoints
- Validation scattered throughout internal code instead of at boundaries
- Breaking changes to existing fields (type changes, removals)
- List endpoints without pagination
- Verbs in REST URLs (`/api/createTask`)
- Third-party API responses used without validation

## Verification

After designing an API:

- [ ] Every endpoint has typed input and output schemas
- [ ] Error responses follow a single consistent format
- [ ] Validation happens at system boundaries only
- [ ] List endpoints support pagination
- [ ] New fields are additive and optional (backward compatible)
- [ ] Naming follows consistent conventions across all endpoints
- [ ] API documentation or types are committed alongside the implementation
