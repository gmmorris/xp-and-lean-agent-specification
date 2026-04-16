---
description: Implement the next task — RED-GREEN-REFACTOR, verify, commit
---

Load the `tdd`, `testing`, and `refactoring` skills.

You are starting the **implementation phase** — building the next task from the plan.

## For Each Task

1. **Read** the task's acceptance criteria from the plan
2. **Load context** — existing code, patterns, types relevant to this task
3. **RED** — Write a failing test for the expected behavior
4. **GREEN** — Implement the minimum code to pass the test
5. **REFACTOR** — Assess improvement opportunities (only if they add clear value)
6. **Verify** — Run the full test suite and build. All green before moving on.
7. **Self-review** — Load the `code-review` skill. Check your own changes before presenting them:
   - No leftover debugging code or TODOs without context
   - No unintended file changes or formatting noise
   - Documentation updated if behavior or API changed
8. **Commit** — Descriptive message explaining why, not just what

## On Failure

If tests fail or behavior is unexpected, load the `debugging-and-error-recovery` skill:
- STOP — don't push past the failure
- REPRODUCE — make it happen reliably
- DIAGNOSE — use the triage checklist
- FIX — root cause, not symptoms
- GUARD — write a regression test

## On Completion

After all tasks are done:
- Run the full test suite one final time
- Run the build
- Self-review the complete changeset via the `code-review` skill
- Update the spec/plan if anything changed during implementation

## Scope Discipline

If you notice improvements outside the current task, record them (NOTICED-BUT-NOT-TOUCHING from `small-safe-steps`) and continue. Don't mix unrelated changes into the current task.
