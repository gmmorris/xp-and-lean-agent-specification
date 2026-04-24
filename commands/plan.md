---
description: Break work into small, verifiable tasks with dependency ordering
---

Load the `planning` skill and the `small-safe-steps` skill.

You are starting the **planning phase** — breaking approved requirements into implementable tasks.

## Step 1: Read Context

Read the existing spec (SPEC.md or equivalent) and relevant codebase sections. Understand:
- What's being built and why
- What already exists
- What constraints apply

## Step 2: Plan

Enter plan mode — read only, no code changes.

1. Identify major components and their dependencies (draw the dependency graph)
2. Determine implementation order — dependencies first, riskiest slice first
3. Identify what can be built in parallel vs. what must be sequential (parallelisation matrix)
4. Note risks and how to mitigate them
5. Apply the step size heuristic from the `planning` skill — if a step touches more than 3 files, it's probably too big

## Step 3: Write Tasks

Break the plan into discrete, implementable tasks. Each task:
- Is completable in a single focused session
- Has explicit acceptance criteria
- Includes a verification step (test command, build, manual check)
- Is ordered by dependency, not by perceived importance
- Represents working software after completion — every commit is deployable

```markdown
- [ ] **[Task title]**
  Acceptance: [What must be true when done]
  Verify: [How to confirm]
  Files: [Which files will be touched]
```

## Step 4: Confirm

Present the plan for review. The user should be able to say "yes, that's the right approach" or "no, change X."

Do NOT start implementation until the user approves the plan.
