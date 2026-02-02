# Engineering Philosophy & Working Agreement for Agents

## We Follow XP and Lean

Software engineering is fundamentally about managing complexity and uncertainty while delivering value. After years of experience, I've found that XP and Lean provide the most effective framework for this because they:

**Optimize for learning and adaptation** rather than prediction. We acknowledge we can't know everything upfront, so we build systems that help us learn fast and change direction cheaply. This beats trying to "get it right" the first time through extensive planning.

**Minimize waste through feedback loops**. The faster we can go from idea → working code → real feedback, the less time we spend building the wrong thing or over-engineering the right thing. Small batches, continuous integration, and TDD aren't bureaucracy—they're how we avoid building bridges to nowhere.

**Respect the human element**. Vibe coding has its place, but we are building systems that scale, and products that deliver value in both the short **and** the long term. Given that context, our code is read 10x more than it's written. Systems outlive their original authors. Sustainable pace beats hero sprints. These methodologies put long-term maintainability and team health on equal footing with delivery speed.

The practices below aren't rules to follow blindly—they're patterns that have emerged from these principles. When in doubt, return to the principles.

### Core Principles

#### Extreme Programming (XP) Values
- **Simplicity**: Do the simplest thing that could possibly work. YAGNI (You Aren't Gonna Need It) is sacred.
- **Communication**: Code is communication. Write for humans first, computers second.
- **Feedback**: Fast feedback loops at every level - tests, CI, deployment, user feedback.
- **Courage**: Refactor mercilessly. Delete code. Challenge assumptions. Admit unknowns.
- **Respect**: For the codebase, for future maintainers, for the users.

#### Lean Thinking
- **Eliminate Waste**: No speculative features. No premature optimization. No gold plating.
- **Amplify Learning**: Prototype, measure, learn. Fail fast and learn faster.
- **Decide as Late as Possible**: Keep options open until we have real data.
- **Deliver as Fast as Possible**: Small batches. Continuous integration. Ship often.
- **Empower the Team**: Decisions at the point of knowledge.
- **Build Quality In**: Testing isn't a phase, it's continuous.
- **See the Whole**: Understand the system, not just the component.

## How I Work

### Starting Any Task
1. **Understand the "why"** before the "how" - What problem are we actually solving? For whom?
2. **Challenge assumptions** - Is this necessary? Can we solve this with less?
3. **Spike if uncertain** - Time-box exploration of unknowns (30min-2hrs max)
4. **Slice vertically** - Break work into end-to-end deliverable increments

## Development Approach

### Preferred Tools:

- **Language**:
    - **Web**: TypeScript (strict mode)
    - **APIs and Business Logic**: Rust or Go
    - **Systems and CLI**: Rust
    - **Data Science**: Python, minimize use where possible
    - **IaC**: Terraform
- **Testing**:
    - **Web**: Vitest + React Testing Library
- **State Management**: Prefer immutable patterns

### Test-Driven Development (TDD)

**TEST-DRIVEN DEVELOPMENT IS NON-NEGOTIABLE.** Every single line of production code must be written in response to a failing test. No exceptions. This is not a suggestion or a preference - it is the fundamental practice that enables all other principles in this document.

I follow Test-Driven Development (TDD) with a strong emphasis on behavior-driven testing and functional programming principles. All work should be done in small, incremental changes that maintain a working state throughout development.

**Core principle**: RED-GREEN-REFACTOR is the rhythm in small, known-good increments. TDD is the fundamental practice.

**Quick reference:**
- RED: Write failing test first (NO production code without failing test)
- GREEN: Write MINIMUM code to pass test
- REFACTOR: Assess improvement opportunities (only refactor if adds value)
- **Wait for commit approval** before every commit
- Each increment leaves codebase in working state
- Capture learnings as they occur, merge at end

For detailed TDD workflow, load the `tdd` skill.
For refactoring methodology, load the `refactoring` skill.
For significant work, load the `planning` skill for three-document model (PLAN.md, WIP.md, LEARNINGS.md).

#### Testing Principles

**Core principle**: Test behavior, not implementation. 100% coverage through business behavior.

**Key Principles:**
- Write tests first (TDD)
- Tests are first-class code - clear, maintainable, fast
- If it's hard to test, the design needs work
- Unit tests for logic, integration tests for behavior, avoid mocking excessively
- Test behavior, not implementation
- Test through public API exclusively
- Use factory functions for test data (no `let`/`beforeEach`)
- Tests must document expected business behavior
- No 1:1 mapping between test files and implementation files

For detailed testing patterns and examples, load the `testing` skill.
For verifying test effectiveness through mutation analysis, load the `mutation-testing` skill.


### Incremental Design
- Start with the simplest design that works
- Let patterns emerge from real needs, don't impose them upfront
- Refactor continuously as understanding grows
- Code duplication is acceptable initially - wait for the third instance before abstracting
- Prefer composition over inheritance, functions over frameworks

#### Code Quality
- **Readability**: Code should read like prose. Names matter immensely.
- **Minimalism**: Fewer lines, fewer files, fewer abstractions when possible
- **Locality**: Related things should be near each other
- **Reversibility**: Avoid decisions that are hard to undo
- **Convention**: Follow existing patterns in the codebase unless there's a compelling reason to change

### Code Style

**Core principle**: Functional programming with immutable data. Self-documenting code.

**Quick reference:**
- No data mutation - immutable data structures only
- Pure functions wherever possible
- No nested if/else - use early returns or composition
- No comments - code should be self-documenting
- Prefer options objects over positional parameters
- Use array methods (`map`, `filter`, `reduce`) over loops

For detailed patterns and examples, load the `functional` skill.

## Workflow Practices

### Continuous Integration
- Integrate to main frequently (multiple times per day if possible)
- Keep the build green - broken builds are stop-the-line events
- Run tests locally before pushing
- Small commits with clear messages explaining "why" not just "what"

### Pair/Mob Programming
- Complex problems benefit from collaboration
- Knowledge sharing is continuous, not a separate activity
- Driver/navigator dynamic - switching regularly

### Working with Uncertainty
- **Known knowns**: Execute with confidence
- **Known unknowns**: Spike, research, ask questions - time-box this
- **Unknown unknowns**: Admit them. Build in feedback loops to surface them early
- When stuck: Take a walk, explain the problem out loud (rubber duck), ask for help

## Code Review Mindset
- Reviews should be quick (< 1 hour turnaround ideally)
- Focus on: correctness, readability, testability, security
- Nitpicks are fine but labeled as such
- "Ask, don't tell" - questions often work better than demands
- Every review is a learning opportunity in both directions

## Communication Style
- **Prefer async communication** with context (don't assume I remember every detail)
- **Surface trade-offs** explicitly - there are no perfect solutions
- **Admit uncertainty** - "I don't know" is a valid and respected answer
- **Show, don't just tell** - code examples, diagrams, prototypes
- **Document decisions**, especially the ones we chose NOT to make

## What I Expect from Agents like OpenCode and Claude Code

**Core principle**: Think deeply, follow TDD strictly, capture learnings while context is fresh.

### When Writing Code
- ALWAYS FOLLOW TDD - no production code without failing test
- Make small, incremental changes
- Assess refactoring after every green (but only if adds value)
- Update AGENTS.md when introducing meaningful changes
- Ask "What do I wish I'd known at the start?" after significant changes
- Document gotchas, patterns, decisions, edge cases while context is fresh
- Explain trade-offs you're making
- Flag areas of uncertainty or technical debt being introduced

### When Problem-Solving
- Ask clarifying questions if the requirement is ambiguous
- Propose the simplest solution first
- Suggest alternatives if you see them
- Challenge the premise if something seems overcomplicated
- Time-box exploratory work and report findings

For detailed TDD workflow, load the `tdd` skill.
For refactoring methodology, load the `refactoring` skill.
For detailed guidance on expectations and documentation, load the `expectations` skill.

### When Stuck
- Say so explicitly
- Show what you've tried
- Suggest next steps for investigation
- Ask specific questions

### Anti-Patterns to Avoid
- ❌ Premature abstraction (wait for the Rule of Three)
- ❌ Over-engineering "for future flexibility"
- ❌ Skipping tests "to move faster"
- ❌ Large, monolithic changes
- ❌ Clever code that's hard to understand
- ❌ Copy-pasting without understanding
- ❌ Adding dependencies without evaluating trade-offs

## Specific Technical Preferences
[Add your language-specific, framework-specific, or tooling preferences here]

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