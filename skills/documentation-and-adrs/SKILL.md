---
name: documentation-and-adrs
description: Records architectural decisions and documents context. Use when making significant technical decisions, changing public APIs, or when you need to record the "why" behind choices for future engineers and agents.
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
---

# Documentation and ADRs

STARTER_CHARACTER = 📝

## Why This Matters

Code shows *what* was built. Documentation explains *why it was built this way* and *what alternatives were considered*. This context is essential for future humans and agents working in the codebase. Without it, decisions get re-litigated, mistakes get repeated, and onboarding takes longer than it should.

The `expectations` skill covers *when* to document and the format for capturing gotchas and learnings. This skill focuses on *what* to document — specifically Architecture Decision Records and inline documentation that captures intent.

## When to Use

- Making a significant architectural decision
- Choosing between competing approaches
- Adding or changing a public API
- When you find yourself explaining the same thing repeatedly

**When NOT to use:** Don't document obvious code. Don't add comments that restate what the code already says. Don't write docs for throwaway prototypes.

## Architecture Decision Records (ADRs)

ADRs capture the reasoning behind significant technical decisions. They're the highest-value documentation you can write — a 10-minute ADR prevents a 2-hour debate about the same decision six months later.

### When to Write an ADR

- Choosing a framework, library, or major dependency
- Designing a data model or database schema
- Selecting an authentication strategy
- Deciding on an API architecture (REST vs. GraphQL vs. tRPC)
- Choosing between build tools, hosting platforms, or infrastructure
- Any decision that would be expensive to reverse

### ADR Template

Store ADRs in `docs/decisions/` with sequential numbering:

```markdown
# ADR-001: [Decision title]

## Status
Accepted | Superseded by ADR-XXX | Deprecated

## Date
YYYY-MM-DD

## Context
What problem are we solving? What constraints exist?
What forces are at play?

## Decision
What did we decide, and what technology/approach did we choose?

## Alternatives Considered

### [Alternative A]
- Pros: ...
- Cons: ...
- Rejected because: ...

### [Alternative B]
- Pros: ...
- Cons: ...
- Rejected because: ...

## Consequences
What are the implications? What are the trade-offs?
What new constraints does this decision create?
```

### ADR Lifecycle

```
PROPOSED → ACCEPTED → (SUPERSEDED or DEPRECATED)
```

- **Don't delete old ADRs.** They capture historical context.
- When a decision changes, write a new ADR that references and supersedes the old one.
- An ADR that was wrong is still valuable — it documents what was tried and why it didn't work.

## Inline Documentation

### Comment the "Why", Not the "What"

```typescript
// BAD: Restates the code
// Increment counter by 1
counter += 1;

// GOOD: Explains non-obvious intent
// Rate limit uses a sliding window — reset counter at window boundary,
// not on a fixed schedule, to prevent burst attacks at window edges
if (now - windowStart > WINDOW_SIZE_MS) {
  counter = 0;
  windowStart = now;
}
```

### When NOT to Comment

- Self-explanatory code (clear function names + types = self-documenting)
- TODO comments for things you should just do now
- Commented-out code (delete it — git has history)

### Document Known Gotchas

When a function has non-obvious constraints, document them at the point where they matter:

```typescript
/**
 * IMPORTANT: This function must be called before the first render.
 * If called after hydration, it causes a flash of unstyled content
 * because the theme context isn't available during SSR.
 *
 * See ADR-003 for the full design rationale.
 */
export function initializeTheme(theme: Theme): void { ... }
```

## Documentation for Agents

AI agents are first-class consumers of documentation. Special consideration:

- **CLAUDE.md / rules files** — Document project conventions so agents follow them
- **Spec files** — Keep specs updated so agents build the right thing
- **ADRs** — Help agents understand why past decisions were made (prevents re-deciding settled questions)
- **Inline gotchas** — Prevent agents from falling into known traps

When writing documentation, ask: "Would an agent working in this codebase for the first time make the right decisions with this context?"

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "The code is self-documenting" | Code shows what. It doesn't show why, what alternatives were rejected, or what constraints apply. |
| "We'll write docs when the API stabilizes" | APIs stabilize faster when you document them. The doc is the first test of the design. |
| "Nobody reads docs" | Agents do. Future engineers do. Your 3-months-later self does. |
| "ADRs are overhead" | A 10-minute ADR prevents a 2-hour debate about the same decision six months later. |
| "Comments get outdated" | Comments on *why* are stable. Comments on *what* get outdated — that's why you only write the former. |

## Red Flags

- Architectural decisions with no written rationale
- Public APIs with no documentation or types
- Commented-out code instead of deletion
- TODO comments that have been there for weeks
- No ADRs in a project with significant architectural choices
- Documentation that restates the code instead of explaining intent

## Verification

After documenting:

- [ ] ADRs exist for all significant architectural decisions
- [ ] API functions have parameter and return type documentation
- [ ] Known gotchas are documented inline where they matter
- [ ] No commented-out code remains
- [ ] Rules files (CLAUDE.md etc.) are current and accurate
