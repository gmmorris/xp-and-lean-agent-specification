# XP and Lean Agent Specification

Agent specifications for AI coding assistants, grounded in XP and Lean principles.

These files configure how AI agents (Claude Code, OpenCode, and similar tools) write code, run tests, review PRs, and communicate. The goal is agents that follow the same engineering discipline you'd expect from a strong team: TDD, small increments, and respectful collaboration.

## Philosophy

Software engineering is about managing complexity and uncertainty while delivering value. XP and Lean provide the framework:

- **Feedback loops over prediction** — TDD, small commits, continuous integration
- **Simplicity over cleverness** — the simplest thing that works, readable by the team
- **Respect for people** — code reviews that are empathetic and culturally aware
- **Eliminate waste** — no over-engineering, no premature abstraction, no bike-shedding

Every specification traces back to these principles.

## How It Works

### AGENTS.md

The central specification. Defines the engineering philosophy, decision-making framework, and references all skills. This is what agents load first.

### CLAUDE.md

Bridges Claude Code to the shared specification. Points Claude at `AGENTS.md` and the skills directory so it follows the same rules as any other agent.

### Skills

Modular, loadable guidelines that agents pull in based on context. Each skill lives in `skills/<name>/SKILL.md`.

| Skill | Purpose |
|-------|---------|
| `tdd` | RED-GREEN-REFACTOR workflow, coverage verification, commit discipline |
| `testing` | Behavior-driven test patterns, test factories, coverage theater detection |
| `test-design-reviewer` | Evaluates test quality using Dave Farley's 8 properties (Farley Score) |
| `mutation-testing` | Find weak or missing tests by analysing if code changes would be caught |
| `test-desiderata` | Analyse test quality against Kent Beck's 12 Test Desiderata properties |
| `code-review` | Culture-aware code review for self-review and peer PR review |
| `refactoring` | When and how to refactor, priority classification, DRY as knowledge |
| `planning` | Outcome-focused user stories, system analysis, CHANGELOG, and small increments |
| `expectations` | Working practices, code quality principles, documentation framework |
| `story-splitting` | Detect oversized stories and split them using proven heuristics |
| `hamburger-method` | Generate layered implementation options and compose minimal vertical slices |
| `small-safe-steps` | Break any work into 1-3h increments with the expand-contract pattern |
| `complexity-review` | Challenge technical proposals against 30 complexity dimensions |
| `debugging-and-error-recovery` | Systematic root-cause debugging with triage checklist and stop-the-line |
| `deprecation-and-migration` | Remove old systems safely with strangler, adapter, and feature flag patterns |
| `source-driven-development` | Ground framework-specific code in official docs, not training data |
| `code-simplification` | Clarity-focused refactoring: reduce complexity while preserving behavior |
| `documentation-and-adrs` | Architecture Decision Records and intent-capturing documentation |
| `api-and-interface-design` | Stable APIs guided by Hyrum's Law and contract-first design |
| `domain-modeling` | Lean domain modeling using DDD techniques for shared language, bounded contexts, and aggregate design |
| `security-and-hardening` | Security-first development with three-tier boundary system |
| `typescript-strict` | TypeScript strict mode patterns, schema-first design, immutability |
| `go` | Go patterns with teaching guidance for engineers coming from Rust/Java/Kotlin |

Skills are loaded on demand — agents pick them up based on what they're doing (writing code, reviewing a PR, planning work).

### Commands

Slash commands that orchestrate skills into workflows. Live in `.claude/commands/`.

| Command | What it does |
|---------|-------------|
| `/discover` | Discover what to build — problem, users, domain language, success criteria |
| `/plan` | Break discovery into epics and tasks with dependency ordering |
| `/build` | Implement the next task: RED-GREEN-REFACTOR, verify, self-review, commit |

The intended workflow is **discover → plan → build**:

1. **`/discover`** — Start here. Work out *what* the system should do from a product perspective. What problem are we solving? For whom? What does success look like? Establish the domain language — name the key concepts, entities, and relationships. This surfaces assumptions, edge cases, and constraints before any implementation thinking.
2. **`/plan`** — Once the discovery is agreed, break the work into epics (coherent slices of domain value) and then tasks within each epic. Domain boundaries guide the slicing — aggregates and bounded contexts are natural epic boundaries.
3. **`/build`** — Pick up the next task and deliver it: write a failing test, make it pass, refactor, self-review, commit. Repeat until the epic is done.

This keeps work flowing in small, lean, valuable increments — each level works at the right abstraction, with domain thinking threaded throughout.

### Sub-Agent Personas

Optional agent personas for specialised review. Live in `agents/`.

| Agent | Role |
|-------|------|
| `test-engineer` | Test strategy, coverage analysis, test quality evaluation |
| `security-auditor` | Vulnerability detection, threat analysis, hardening recommendations |
| `code-reviewer` | Thorough code review across correctness, design, testability, security, readability |

## Setup

Symlink from this repo into your agent config directories. This keeps everything in one place — changes to the repo are picked up immediately.

```sh
git clone <repo-url>
cd xp-and-lean-agent-specification

# For Claude Code
ln -sf "$PWD/AGENTS.md" ~/.claude/CLAUDE.md
ln -sf "$PWD/skills" ~/.claude/skills
ln -sf "$PWD/commands" ~/.claude/commands
ln -sf "$PWD/agents" ~/.claude/agents

# For OpenCode or shared agent config
ln -sf "$PWD/AGENTS.md" ~/.config/opencode/AGENTS.md
ln -sf "$PWD/skills" ~/.config/opencode/skills

# For Gemini
ln -sf "$PWD/AGENTS.md" ~/.gemini/GEMINI.md
ln -sf "$PWD/skills" ~/.gemini/skills
```

Adapt paths to your setup. The key requirement is that `AGENTS.md` can reference the skills directory. The `commands` symlink gives you `/discover`, `/plan`, and `/build` slash commands; the `agents` symlink enables the `test-engineer`, `security-auditor`, and `code-reviewer` sub-agents.

## Attribution

This specification is adapted from [Paul Hummond's .dotfiles](https://github.com/citypaul/.dotfiles). The `test-design-reviewer` skill is adapted from [Andrea Laforgia's claude-code-agents](https://github.com/andlaf-ak/claude-code-agents).

The following skills are adapted from [Lada Kesseler's Skill Factory](https://github.com/lada-k/skill-factory) (Apache License 2.0): `mutation-testing`, `test-desiderata`, `story-splitting`, `hamburger-method`, `small-safe-steps`, and `complexity-review`. The Hamburger Method is by Gojko Adzic. The Small Safe Steps expand-contract pattern and the 30 Complexity Dimensions are by Eduardo Ferro. See NOTICE for full copyright details.

The following are adapted from [Addy Osmani's agent-skills](https://github.com/addyosmani/agent-skills) (MIT License): nuggets merged into existing skills (test-double hierarchy and Test Pyramid in `testing`, the Prove-It Pattern in `tdd`, Chesterton's Fence in `refactoring`, Change Sizing & Splitting/Change Descriptions/Verification Story in `code-review`, dependency graph and parallelisation matrix in `planning`, Risk-First Sequencing and NOTICED-BUT-NOT-TOUCHING in `small-safe-steps`), skill-local reference checklists (security, performance, accessibility, testing patterns), seven new skills adapted to XP/Lean voice (`debugging-and-error-recovery`, `deprecation-and-migration`, `source-driven-development`, `code-simplification`, `documentation-and-adrs`, `api-and-interface-design`, `security-and-hardening`), three commands (`/spec`, `/plan`, `/build`), and three sub-agent personas (`test-engineer`, `security-auditor`, `code-reviewer`).

Thank you to Paul, Andrea, Lada, Gojko, Eduardo, and Addy for creating and sharing these excellent practices.