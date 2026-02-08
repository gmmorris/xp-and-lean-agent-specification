# Agent Configuration

## Project Type Detection

Parse the project's CLAUDE.md front matter (YAML between `---` markers) to determine behavior.

### Front Matter Schema

| Field | Values | Behavior |
|-------|--------|----------|
| `type` | `knowledge-repository` | Second brain mode. Skip coding rules below. Load project's AGENTS.md instead. |
| `type` | `coding` (or absent) | Coding project. Follow XP/Lean practices below. |
| `role` | e.g., `second-brain` | Describes the agent's purpose |
| `skills` | list | Project-specific skills to load from project's `skills/` |

### Default Behavior (no front matter or `type: coding`)

Follow XP and Lean development practices defined below.

---

# Engineering Philosophy & Working Agreement

## Why We Follow XP and Lean

Software engineering is fundamentally about managing complexity and uncertainty while delivering value. After years of experience, I've found that XP and Lean provide the most effective framework for this because they:

**Optimize for learning and adaptation** rather than prediction. We acknowledge we can't know everything upfront, so we build systems that help us learn fast and change direction cheaply. This beats trying to "get it right" the first time through extensive planning.

**Minimize waste through feedback loops**. The faster we can go from idea → working code → real feedback, the less time we spend building the wrong thing or over-engineering the right thing. Small batches, continuous integration, and TDD aren't bureaucracy—they're how we avoid building bridges to nowhere.

**Respect the human element**. Code is read 10x more than it's written. Systems outlive their original authors. Sustainable pace beats hero sprints. These methodologies put long-term maintainability and team health on equal footing with delivery speed.

**These are baseline expectations.** When you see references to XP (Extreme Programming) and Lean Software Development throughout this document, I'm referring to their core values and principles. If you're unfamiliar with these methodologies, research them - they're fundamental to how we work.

The practices below aren't rules to follow blindly—they're patterns that have emerged from these principles. When in doubt, return to the principles.

## Working with Skills

This AGENTS.md provides high-level guidance. For detailed workflows, patterns, and examples, load the relevant skill:

**When to load skills:**

| Situation | Load These Skills |
|-----------|------------------|
| Starting any code work | `tdd`, `testing`, `refactoring` |
| Writing TypeScript | `typescript-strict` |
| Writing Go | `go` |
| Starting significant work or a new feature | `planning` |
| Understanding expectations and working practices | `expectations` |
| Reviewing test quality | `test-design-reviewer` |
| Self-reviewing before presenting work as complete | `code-review` |
| Reviewing a colleague's PR or code changes | `code-review` |

**Core workflow skills:**
- `tdd` - Detailed TDD workflow with examples and common pitfalls
- `testing` - Testing patterns, factory functions, and antipatterns
- `test-design-reviewer` - Framework for reviewing test design quality
- `code-review` - Culture-aware code review for self-review and peer PR review
- `refactoring` - Safe refactoring methodology and techniques
- `planning` - Outcome-focused user stories, system analysis, acceptance criteria, and small increments
- `expectations` - Working practices, agent guidance, code quality principles, and documentation

**Language specific skills:**
- `typescript-strict` - Detailed TypeScript strict mode guidelines and patterns
- `go` - Go patterns and teaching guidance for experienced engineers learning Go

Skills contain detailed examples that this document intentionally omits to stay focused on principles.

## How I Work

### Starting Any Task
1. **Understand the "why"** before the "how" - What problem are we actually solving? For whom?
2. **Challenge assumptions** - Is this necessary? Can we solve this with less?
3. **Spike if uncertain** - Limit exploration to 3-5 focused attempts, then report findings and propose next steps
4. **Slice vertically** - Break work into end-to-end deliverable increments

## Development Approach

### Preferred Tools

- **Language**:
    - **Web**: TypeScript (strict mode)
    - **APIs and Business Logic**: Rust or Go
    - **Systems and CLI**: Rust
    - **IaC**: Terraform
- **Testing**:
    - **Web**: Vitest + React Testing Library
- **State Management**: Prefer immutable patterns

#### TypeScript Guidelines

**Core principle**: Strict mode always. Schema-first at trust boundaries, types for internal logic.

**Quick reference:**
- No `any` types - ever (use `unknown` if type truly unknown)
- No type assertions without justification
- Define schemas first, derive types from them (Zod/Standard Schema)
- Use schemas at trust boundaries, plain types for internal logic

For detailed TypeScript patterns and rationale, load the `typescript-strict` skill.

#### Go Guidelines

**Core principle**: Write clear, idiomatic Go. Bring Rust-quality thinking (explicit errors, ownership discipline, composition) but respect Go's idioms.

**Quick reference:**
- Handle every error — never discard with `_`, always wrap with context
- Accept interfaces, return structs — keep interfaces small (1-2 methods)
- Organize by domain, not by technical layer — no `util` or `common` packages
- Start synchronous — add goroutines only when there's a clear need

For detailed Go patterns, Rust-to-Go concept bridges, and teaching guidance, load the `go` skill.

## Core Non-Negotiables

### Test-Driven Development

**TEST-DRIVEN DEVELOPMENT IS NON-NEGOTIABLE.** Every single line of production code must be written in response to a failing test. No exceptions. This is the fundamental practice that enables all other principles in this document.

Follow the RED-GREEN-REFACTOR cycle in small, known-good increments. For detailed TDD workflow, examples, and common pitfalls, load the `tdd` skill.

### TypeScript Strict Mode

**Strict mode always. No `any` types, ever.**

For detailed patterns and guidelines, load the `typescript-strict` skill.

### Working with Uncertainty

When facing uncertainty, don't guess or make assumptions:

**Known unknowns** - Spike and research, but time-box to 3-5 focused attempts, then report findings and propose next steps.

**When stuck** - State explicitly what's blocking you, show what you've tried, propose 2-3 alternative paths forward.

**Never proceed on assumptions.** Ask clarifying questions instead.

For detailed problem-solving approaches, load the `expectations` skill.

### Documentation & Communication

**Keep documentation current.** After any significant change:
- Update relevant docs (AGENTS.md, READMEs, etc.)
- Document learnings: gotchas discovered, patterns emerged, decisions made
- Ask: "What do I wish I'd known at the start?"

**Surface trade-offs explicitly.** Explain the reasoning behind significant decisions and the alternatives considered.

For documentation framework and formats, load the `expectations` skill.

## Decision-Making Framework

When facing technical decisions, consider in this order:
1. **Does it solve the actual problem?** (not a hypothetical future one)
2. **Is it the simplest thing that could work?**
3. **Can we test it easily?**
4. **Can we reverse it if we're wrong?**
5. **Does it align with existing patterns in the codebase?**
6. **Is the complexity justified by the value?**

If the answer to 1 & 2 is yes, and 3-6 don't raise red flags, proceed.

## Summary

The key is to write clean, testable, functional code that evolves through small, safe increments. Every change should be driven by a test that describes the desired behavior, and the implementation should be the simplest thing that makes that test pass. When in doubt, favor simplicity and readability over cleverness.

---

*"Make it work, make it right, make it fast" - in that order, and stop when you get to "good enough".*

*"Make the change easy, then make the easy change" - if the change is hard, refactor until it is no longer hard, then make the change.*