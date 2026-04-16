---
description: Write a structured specification before writing code
---

Load the `planning` skill and the `expectations` skill.

If the work involves APIs or public interfaces, also load `api-and-interface-design`.
If security is a primary concern, also load `security-and-hardening`.

You are starting the **specification phase** — defining what we're building before any code gets written.

## Step 1: Understand

Ask clarifying questions until requirements are concrete. Cover:
1. What are we building and why? Who is the user?
2. What does success look like? (specific, testable conditions)
3. Tech stack, constraints, and known boundaries
4. What's explicitly out of scope?

## Step 2: Surface Assumptions

Before writing any spec content, list what you're assuming:

```
ASSUMPTIONS:
1. [assumption]
2. [assumption]
→ Correct me now or I'll proceed with these.
```

Don't silently fill in ambiguous requirements. Follow the Requirement Ambiguity Protocol — present options, don't assume.

## Step 3: Write the Spec

Generate a structured spec covering:

1. **Objective** — What, why, for whom, success criteria
2. **Tech Stack** — Framework, language, key dependencies with versions
3. **Commands** — Full executable build/test/lint/dev commands
4. **Project Structure** — Directory layout with descriptions
5. **Code Style** — One real code snippet showing the style
6. **Testing Strategy** — Framework, test locations, coverage expectations, test levels
7. **Boundaries** — Always do / Ask first / Never do

Reframe vague requirements as testable success criteria:

```
REQUIREMENT: "Make the dashboard faster"

SUCCESS CRITERIA:
- Dashboard LCP < 2.5s on 4G connection
- Initial data load < 500ms
- No layout shift during load (CLS < 0.1)
→ Are these the right targets?
```

## Step 4: Confirm

Save the spec as `SPEC.md` (or the location the user prefers) and confirm before proceeding. The spec is a living document — it stays in version control and gets updated as decisions change.

Do NOT proceed to planning or implementation until the user approves the spec.
