---
name: rust-bootstrap
description: Bootstraps a new Rust project from scratch through structured architectural interviews, then scaffolds a git repository with incremental commits. Use when creating, initialising, or starting a new Rust project or service.
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
---

# Rust Bootstrap

STARTER_CHARACTER = 🏗️

Creates a new Rust project in three phases: architectural interview, naming, repository generation. Do not skip or collapse phases.

## Phase 1: Architecture Interview

Read and run the interview in [references/interview.md](references/interview.md).

Cover every dimension. Do not infer answers — ask. After all questions are answered, present a confirmation table and wait for explicit approval before proceeding.

## Phase 2: Project Naming

If the user has not provided a name:
- Ask for the project's purpose in one sentence
- Suggest three candidate names (lowercase, hyphens only, no reserved words)
- Confirm the chosen name — this becomes the crate name, binary name, and directory name

## Phase 3: Repository Generation

Follow the commit sequence in [references/commit-sequence.md](references/commit-sequence.md).

Work in the current directory, or create a new subdirectory if one was not provided. Run `cargo build && cargo test` after each commit. Fix failures before moving to the next commit.

**TDD applies to the scaffolding itself.** Infrastructure commits that add observable behaviour — the HTTP server, the OTel pipeline, the metrics endpoint — must be accompanied by tests that verify that behaviour works. The E2E test commit is not just a health check; it includes tests that assert OTel is emitting traces and the metrics endpoint returns valid output. A scaffold with no tests verifying its own foundations is not done.

Announce the final structure and the command to start development when generation is complete.

## Dependency Rules

- Always use `cargo add <crate>` — never hand-write versions into `Cargo.toml`
- Never specify version numbers in generated `Cargo.toml` snippets or instructions
- Where crates must stay version-compatible with each other (e.g. the OTel ecosystem), note the constraint in prose — do not pin numbers

## Structural Rules

These apply to all generated code:

- `src/lib.rs` contains domain logic — zero Axum, SQLx, or other framework imports
- `src/main.rs` is the binary entry point — declares binary-only modules
- Business logic lives in plain synchronous functions, not in async handlers
- Errors are plain enums with `#[derive(Debug, PartialEq)]`
- Configuration comes from environment variables + `.env` file via `dotenvy`
- No `util`, `helpers`, or `common` modules — organise by domain

## Anti-Patterns

- Generating any code before the interview confirmation is received
- Multiple concerns in one commit
- Business logic inside async handler functions
- Skipping `cargo build && cargo test` between commits
- Creating files the interview did not call for
