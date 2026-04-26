---
description: Implement the next task — RED-GREEN-REFACTOR, verify, commit
---

Load the `tdd`, `testing`, and `refactoring` skills.

You are starting the **implementation phase** — building the next task from the plan.

## Before You Write Any Code

Load the `grill-me` skill. Before touching any code, interview me about this task. Walk through every branch of the design: what we're building, why, edge cases, trade-offs, and anything that could go wrong. Resolve each question before moving on to the next. Only proceed to implementation once we've reached shared understanding.

Once the interview is complete:
1. **Update the plan** with any decisions, constraints, or changes that emerged from the discussion.
2. **Record architectural decisions** — Load the `documentation-and-adrs` skill. If any answers revealed significant design choices, trade-offs, or rejected alternatives, capture them in the relevant ADR(s). Create a new ADR if none exists for the decision area.

## Critically Important

1. Follow all our testing best practices.
2. Lay out our implementation as a detailed step by step - the aim being that each step is a small vertical slice of the implementation, representing a clean commit point where the software is in working state, and has all the necessary tests (unit -> e2e) because - of course- you have followed our TDD rules
3. **Never commit without my explicit approval** - always ask and wait for my confirmation before committing. Even when all checks pass and the code is ready, present the changeset and wait. When I approve, use `git duet-commit` to ensure all collaborators are properly attributed in the commit history.

## For Each Step

1. **Read** the task's acceptance criteria from the plan
2. **Load context** — existing code, patterns, types relevant to this task
3. **RED** — Write a failing test for the expected behavior
4. **GREEN** — Implement the minimum code to pass the test
5. **REFACTOR** — Assess improvement opportunities (only if they add clear value)
6. **Verify** — Run the full test suite and build. All green before moving on.
7. **Mutation test** — Load the `mutation-testing` skill. Analyse the tests written in this slice to verify they would catch real bugs. If mutations survive, strengthen the tests before proceeding.
8. **Self-review** — Load the `code-review` skill. Check your own changes before presenting them:
   - No leftover debugging code or TODOs without context
   - No unintended file changes or formatting noise
   - Documentation updated if behavior or API changed
9. **Present for commit** — Descriptive message explaining why, not just what. **Wait for my explicit approval before committing.**

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
- **Present the final changeset and wait for my explicit approval before committing**

## Scope Discipline

If you notice improvements outside the current task, record them (NOTICED-BUT-NOT-TOUCHING from `small-safe-steps`) and continue. Don't mix unrelated changes into the current task.
