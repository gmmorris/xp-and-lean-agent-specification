---
description: Break an approved spec into epics and tasks with dependency ordering
---

Load the `planning` skill, the `small-safe-steps` skill, and the `domain-modeling` skill.

You are starting the **planning phase** — breaking an approved spec into deliverable increments.

## Step 1: Read Context

Read the discovery document (SPEC.md or equivalent) and relevant codebase sections. Understand:
- What's being built and why
- The domain concepts and language established in the spec
- What already exists in the codebase
- What constraints apply

## Step 2: Identify Epics

Enter plan mode — read only, no code changes.

Break the spec into **epics** — coherent slices of domain value. Each epic should:
- Deliver a meaningful, end-to-end capability
- Align with domain boundaries (aggregates, bounded contexts, natural seams in the model)
- Have its own acceptance criteria drawn from the spec's success criteria
- Be independently deployable and valuable

Use the domain model to guide the slicing. Aggregates and bounded contexts are natural epic boundaries — work that crosses them is a sign the epic is too broad.

**Map epic dependencies before ordering:**

```
Epic 1: Organisation management
    │
    ├── Epic 2: Site management (needs orgs from #1)
    │       │
    │       └── Epic 3: Geography management (needs sites from #2)
    │
    └── Epic 4: Reporting (independent of #2/#3 — can run in parallel)
```

Order epics so each one only depends on epics already delivered. Identify what can run in parallel vs. what must be sequential.

## Step 3: Break Epics into Tasks

For each epic, decompose into discrete, implementable tasks. Each task:
- Is completable in a single focused session
- Has explicit acceptance criteria
- Includes a verification step (test command, build, manual check)
- Is ordered by dependency within the epic
- Represents working software after completion — every commit is deployable

```markdown
- [ ] **[Task title]**
  Acceptance: [What must be true when done]
  Verify: [How to confirm]
  Files: [Which files will be touched]
```

Apply the step size heuristic: if a task touches more than 3 files, it's probably too big. If you can't describe it in one sentence, break it down further.

## Step 4: Confirm

Present the plan for review. The user should be able to see:
- The epics and how they relate to each other
- The tasks within each epic and their ordering
- Where the domain boundaries informed the slicing

Do NOT start implementation until the user approves the plan.
