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

**Core workflow skills:**
- `tdd` - Detailed TDD workflow with examples and common pitfalls
- `testing` - Testing patterns, factory functions, and antipatterns
- `test-design-reviewer` - Framework for reviewing test design quality
- `refactoring` - Safe refactoring methodology and techniques
- `planning` - Three-document model (PLAN.md, WIP.md, LEARNINGS.md) for significant work
- `expectations` - Best practices for capturing learnings and documenting code changes

**Language specific skills:**
- `typescript-strict` - Detailed TypeScript strict mode guidelines and patterns

**When to load skills:**
- Load Language specific skills when working in that language
- Load `tdd`, `testing` and `refactoring` at the start of any coding work
- Load `planning` when starting work that involves significant design or uncertainty
- Load other skills as needed for specific guidance

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

### Test-Driven Development (TDD)

**TEST-DRIVEN DEVELOPMENT IS NON-NEGOTIABLE.** Every single line of production code must be written in response to a failing test. No exceptions. This is not a suggestion or a preference - it is the fundamental practice that enables all other principles in this document.

We follow Test-Driven Development (TDD) with a strong emphasis on behavior-driven testing and functional programming principles. All work should be done in small, iterative changes that maintain a working state throughout development.

**Core principle**: RED-GREEN-REFACTOR is the rhythm in small, known-good increments. TDD is the fundamental practice.

**The TDD Cycle:**

1. **RED** - Write a failing test
   - Write the test that describes the behavior you want
   - The test MUST fail (if it passes, you're not testing new behavior)
   - Do NOT write any production code yet

2. **GREEN** - Make it pass with minimal code
   - Write ONLY enough code to make the test pass
   - Resist the urge to write "just a bit more"
   - Prefer obvious/simple over clever/elegant at this stage

3. **REFACTOR** - Improve the design
   - Look for duplication, unclear names, awkward structure
   - Only refactor if it adds clear value
   - All tests must still pass after refactoring

4. **COMMIT** - Confirm readiness before committing
   - All tests pass?
   - System in working state?
   - Code is clean and readable?
   - Each commit represents a complete, working increment

**For detailed TDD workflow including examples and common pitfalls, load the `tdd` skill.**

#### Testing Principles

**Test behavior, not implementation.** Aim for 100% coverage through business behavior, not line coverage metrics.

**Key practices:**
- Tests are first-class code - clear, maintainable, fast
- If it's hard to test, the design needs work
- Unit tests for logic, integration tests for behavior
- Avoid excessive mocking - test through real collaborators when possible
- Use factory functions for test data (no `let`/`beforeEach`)
- Tests must document expected business behavior
- No 1:1 mapping between test files and implementation files
- Test through public API exclusively

**For detailed testing patterns, antipatterns, and examples, load the `testing` skill.**

### Incremental Design
- Start with the simplest design that works
- Let patterns emerge from real needs, don't impose them upfront
- Refactor continuously as understanding grows (but only when value is clear)
- Code duplication is acceptable initially - wait for the third instance before abstracting (Rule of Three)
- Prefer composition over inheritance, functions over frameworks

### Code Quality
- **Readability**: Code should read like prose. Names matter immensely.
- **Minimalism**: Fewer lines, fewer files, fewer abstractions when possible
- **Locality**: Related things should be near each other
- **Reversibility**: Avoid decisions that are hard to undo
- **Convention**: Follow existing patterns in the codebase unless there's a compelling reason to change

### Code Style

**Functional programming with immutable data. Self-documenting code.**

**Core practices:**
- No data mutation - immutable data structures only
- Pure functions wherever possible
- No nested if/else - use early returns or composition
- Avoid comments - code should be self-documenting through clear naming
- Prefer options objects over positional parameters
- Use array methods (`map`, `filter`, `reduce`) over loops

## Workflow Practices

### Continuous Integration
- Integrate to main frequently (multiple times per day if possible)
- Keep the build green - broken builds are stop-the-line events
- Run tests locally before committing and pushing
- Small commits with clear messages explaining "why" not just "what"

### Working with Uncertainty
- **Known knowns**: Execute with confidence
- **Known unknowns**: Spike, research, ask questions - limit to 3-5 focused attempts
- **Unknown unknowns**: Admit them. Build in feedback loops to surface them early
- When stuck: Pause, explain the problem out loud (rubber duck), ask for help

## Code Review Mindset
- Reviews should be quick and focused
- Focus on: correctness, readability, testability, security
- Nitpicks are fine but labeled as such
- "Ask, don't tell" - questions often work better than demands
- Every review is a learning opportunity in both directions

## Communication Style
- **Keep project docs current** - Update docs when introducing changes
- **Surface trade-offs** explicitly - there are no perfect solutions
- **Admit uncertainty** - "I don't know" is a valid and respected answer
- **Show, don't just tell** - code examples, diagrams, prototypes
- **Document decisions**, especially the ones we chose NOT to make

## What I Expect from Agents

**Core principle**: Think deeply, follow TDD strictly, capture learnings while context is fresh.

### When Writing Code
- ALWAYS follow TDD - no production code without failing test first (load the `tdd` skill)
- Make small, incremental changes that leave the system in a working state
- Assess refactoring after every green (but only if it adds clear value)
- Update change logs when introducing any change
- Update documentation when introducing meaningful patterns or changes
- After significant changes, document: gotchas discovered, patterns that emerged, decisions made and why
- Ask "What do I wish I'd known at the start?" after significant changes
- Explain trade-offs you're making
- Flag areas of uncertainty or technical debt being introduced

### When Problem-Solving
- Ask clarifying questions if the requirement is ambiguous
- Propose the simplest solution first
- Suggest alternatives if you see them
- Challenge the premise if something seems overcomplicated
- Time-box exploratory work and report findings

### When Stuck
- State explicitly what's blocking you
- Show what you've tried (list specific approaches)
- Propose 2-3 alternative paths forward
- Ask specific questions about trade-offs or unknowns

### Anti-Patterns to Avoid
- ❌ Writing production code before writing a failing test
- ❌ Premature abstraction (wait for the Rule of Three)
- ❌ Over-engineering "for future flexibility"
- ❌ Skipping tests "to move faster"
- ❌ Large, monolithic changes
- ❌ Clever code that's hard to understand
- ❌ Copy-pasting without understanding
- ❌ Adding dependencies without evaluating trade-offs

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