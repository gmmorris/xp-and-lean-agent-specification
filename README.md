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
| `code-review` | Culture-aware code review for self-review and peer PR review |
| `refactoring` | When and how to refactor, priority classification, DRY as knowledge |
| `planning` | Three-document model (PLAN.md, WIP.md, LEARNINGS.md) for significant work |
| `expectations` | Working practices, code quality principles, documentation framework |
| `typescript-strict` | TypeScript strict mode patterns, schema-first design, immutability |
| `go` | Go patterns with teaching guidance for engineers coming from Rust/Java/Kotlin |

Skills are loaded on demand — agents pick them up based on what they're doing (writing code, reviewing a PR, planning work).

## Setup

Symlink from this repo into your agent config directories. This keeps everything in one place — changes to the repo are picked up immediately.

```sh
git clone <repo-url>
cd xp-and-lean-agent-specification

# For Claude Code
ln -sf "$PWD/CLAUDE.md" ~/.claude/CLAUDE.md
ln -sf "$PWD/skills" ~/.claude/skills

# For OpenCode or shared agent config
ln -sf "$PWD/AGENTS.md" ~/.config/opencode/AGENTS.md
ln -sf "$PWD/skills" ~/.config/opencode/skills
```

Adapt paths to your setup. The key requirement is that `AGENTS.md` can reference the skills directory.

## Attribution

This specification is adapted from [Paul Hummond's .dotfiles](https://github.com/citypaul/.dotfiles). The `test-design-reviewer` skill is adapted from [Andrea Laforgia's claude-code-agents](https://github.com/andlaf-ak/claude-code-agents).

Thank you to Paul and Andrea for creating and sharing these excellent specifications.