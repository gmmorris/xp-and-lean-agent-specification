---
description: Discover what to build — problem, users, domain language, success criteria
---

Load the `planning` skill, the `expectations` skill, and the `domain-modeling` skill.

If the work involves APIs or public interfaces, also load `api-and-interface-design`.
If security is a primary concern, also load `security-and-hardening`.

You are starting the **discovery phase** — understanding *what* we're building and *why* before any implementation thinking.

## Step 1: Understand the Problem

Ask clarifying questions until the problem and its context are concrete. Cover:
1. **Who is this for?** — which user, in which role, in which context?
2. **What problem are we solving?** — what's painful, missing, or broken today?
3. **What does success look like?** — specific, testable outcomes
4. **What constraints exist?** — technical, organisational, regulatory, timeline
5. **What's explicitly out of scope?**

Don't accept a solution as a requirement. "Add a caching layer" is an implementation. "Users should see results within 200ms" is a requirement. Push for the requirement.

## Step 2: Discover the Domain

Use the domain modeling workflow to establish shared language:

1. **Identify domain concepts** — list the nouns (entities, value objects) and verbs (operations, events) from conversations with the user
2. **Build a glossary** — document each term with a clear definition, examples, and invariants
3. **Map relationships** — how do these concepts relate? What are the boundaries?
4. **Identify bounded contexts** — where does the same term mean different things? Where are the natural seams?

The goal is a shared vocabulary that will carry through planning and implementation. If we can't name the domain concepts clearly, we don't understand the problem well enough.

## Step 3: Surface Assumptions

Before writing any spec content, list what you're assuming:

```
ASSUMPTIONS:
1. [assumption]
2. [assumption]
→ Correct me now or I'll proceed with these.
```

Don't silently fill in ambiguous requirements. Follow the Requirement Ambiguity Protocol — present options, don't assume.

## Step 4: Write the Spec

Generate a structured spec covering:

1. **Problem Statement** — what's broken or missing, and for whom
2. **Users and Roles** — who interacts with this, in what capacity
3. **Domain Concepts** — glossary of key terms, entities, value objects, and their relationships (use diagrams where they add clarity)
4. **Success Criteria** — specific, testable conditions that confirm the problem is solved
5. **Constraints** — technical, organisational, regulatory, or timeline boundaries
6. **Out of Scope** — what we're explicitly not doing

Reframe vague requirements as testable success criteria:

```
REQUIREMENT: "Make the dashboard faster"

SUCCESS CRITERIA:
- Dashboard LCP < 2.5s on 4G connection
- Initial data load < 500ms
- No layout shift during load (CLS < 0.1)
→ Are these the right targets?
```

## Step 5: Confirm

Save the discovery as `DISCOVERY.md` (or the location the user prefers) and confirm before proceeding. The discovery document is a living artefact — it stays in version control and gets updated as understanding evolves.

Do NOT proceed to planning or implementation until the user approves the discovery.
