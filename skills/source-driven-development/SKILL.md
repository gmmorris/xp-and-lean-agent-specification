---
name: source-driven-development
description: Grounds every implementation decision in official documentation. Use when building with any framework or library where correctness matters. Prevents outdated patterns from training data by verifying against authoritative sources.
allowed-tools:
  - Bash
  - Read
  - Glob
  - Grep
  - Edit
  - Write
  - WebFetch
  - WebSearch
  - AskUserQuestion
---

# Source-Driven Development

STARTER_CHARACTER = 📚

## Why This Matters

Training data goes stale. APIs get deprecated. Best practices evolve. An agent implementing from memory will confidently produce code that *looks* correct but uses patterns from two versions ago. This skill ensures the user gets code they can trust because every pattern traces back to an authoritative source they can check.

This is the XP value of *communication* applied to the agent-human relationship: show your sources so the human can verify your work. It's also *building quality in* — catching outdated patterns at implementation time is cheaper than debugging them after deployment.

## When to Use

- Building with any framework or library where the recommended patterns are version-dependent
- Creating boilerplate, starter code, or patterns that will be copied across a project
- Implementing features where the framework's recommended approach matters (forms, routing, data fetching, state management, auth)
- Reviewing or improving code that uses framework-specific patterns
- Any time you are about to write framework-specific code from memory

**When NOT to use:**

- Correctness does not depend on a specific version (renaming variables, fixing typos, moving files)
- Pure logic that works the same across all versions (loops, conditionals, data structures)
- The user explicitly wants speed over verification ("just do it quickly")

## The Process

```
DETECT ──→ FETCH ──→ IMPLEMENT ──→ CITE
  │          │           │            │
  ▼          ▼           ▼            ▼
 What       Get the    Follow the   Show your
 stack?     relevant   documented   sources
            docs       patterns
```

### Step 1: Detect Stack and Versions

Read the project's dependency file to identify exact versions:

```
package.json    → Node/React/Vue/Angular/Svelte
composer.json   → PHP/Symfony/Laravel
requirements.txt / pyproject.toml → Python/Django/Flask
go.mod          → Go
Cargo.toml      → Rust
Gemfile         → Ruby/Rails
```

State what you found explicitly:

```
STACK DETECTED:
- React 19.1.0 (from package.json)
- Vite 6.2.0
- Tailwind CSS 4.0.3
→ Fetching official docs for the relevant patterns.
```

If versions are missing or ambiguous, **ask the user**. Don't guess — the version determines which patterns are correct.

### Step 2: Fetch Official Documentation

Fetch the specific documentation page for the feature you're implementing. Not the homepage, not the full docs — the relevant page.

**Source hierarchy (in order of authority):**

| Priority | Source | Example |
|----------|--------|---------|
| 1 | Official documentation | react.dev, docs.djangoproject.com, doc.rust-lang.org |
| 2 | Official blog / changelog | react.dev/blog, blog.rust-lang.org |
| 3 | Web standards references | MDN, web.dev, html.spec.whatwg.org |
| 4 | Runtime compatibility | caniuse.com, node.green |

**Not authoritative — never cite as primary sources:**

- Stack Overflow answers
- Blog posts or tutorials (even popular ones)
- AI-generated documentation or summaries
- Your own training data (that is the whole point — verify it)

**Be precise with what you fetch:**

```
BAD:  Fetch the React homepage
GOOD: Fetch react.dev/reference/react/useActionState

BAD:  Search "django authentication best practices"
GOOD: Fetch docs.djangoproject.com/en/5.2/topics/auth/
```

After fetching, extract the key patterns and note any deprecation warnings or migration guidance.

When official sources conflict with each other (e.g., a migration guide contradicts the API reference), surface the discrepancy to the user and verify which pattern actually works against the detected version.

### Step 3: Implement Following Documented Patterns

Write code that matches what the documentation shows:

- Use the API signatures from the docs, not from memory
- If the docs show a new way to do something, use the new way
- If the docs deprecate a pattern, don't use the deprecated version
- If the docs don't cover something, flag it as unverified

**When docs conflict with existing project code:**

Surface the conflict. Don't silently pick one. Present options and let the user choose — this follows the Requirement Ambiguity Protocol from our working agreement.

```
CONFLICT DETECTED:
The existing codebase uses [old pattern],
but the current docs recommend [new pattern].
(Source: [URL])

Options:
A) Use the modern pattern — consistent with current docs
B) Match existing code — consistent with codebase
→ Which approach do you prefer?
```

### Step 4: Cite Your Sources

Every framework-specific pattern gets a citation. The user must be able to verify every decision.

**In code comments** (only for non-obvious decisions):

```
// React 19 form handling with useActionState
// Source: https://react.dev/reference/react/useActionState#usage
```

**In conversation** — include URL and quote the relevant passage when it supports a non-obvious decision.

**Citation rules:**

- Full URLs, not shortened
- Prefer deep links with anchors where possible — anchors survive doc restructuring better than top-level pages
- Quote the relevant passage when it supports a non-obvious decision
- Include runtime support data when recommending platform features
- If you cannot find documentation for a pattern, say so explicitly:

```
UNVERIFIED: I could not find official documentation for this
pattern. This is based on training data and may be outdated.
Verify before using in production.
```

Honesty about what you couldn't verify is more valuable than false confidence.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "I'm confident about this API" | Confidence is not evidence. Training data contains outdated patterns that look correct but break against current versions. Verify. |
| "Fetching docs wastes tokens" | Hallucinating an API wastes more. The user debugs for an hour, then discovers the function signature changed. One fetch prevents hours of rework. |
| "The docs won't have what I need" | If the docs don't cover it, that's valuable information — the pattern may not be officially recommended. |
| "I'll just mention it might be outdated" | A disclaimer doesn't help. Either verify and cite, or clearly flag it as unverified. Hedging is the worst option. |
| "This is a simple task, no need to check" | Simple tasks with wrong patterns become templates. The user copies your deprecated form handler into ten components before discovering the modern approach exists. |

## Red Flags

- Writing framework-specific code without checking the docs for that version
- Using "I believe" or "I think" about an API instead of citing the source
- Implementing a pattern without knowing which version it applies to
- Citing Stack Overflow or blog posts instead of official documentation
- Using deprecated APIs because they appear in training data
- Not reading the dependency file before implementing
- Delivering code without source citations for framework-specific decisions

## Verification

After implementing with source-driven development:

- [ ] Framework and library versions were identified from the dependency file
- [ ] Official documentation was fetched for framework-specific patterns
- [ ] All sources are official documentation, not blog posts or training data
- [ ] Code follows the patterns shown in the current version's documentation
- [ ] Non-trivial decisions include source citations with full URLs
- [ ] No deprecated APIs are used (checked against migration guides)
- [ ] Conflicts between docs and existing code were surfaced to the user
- [ ] Anything that could not be verified is explicitly flagged as unverified
