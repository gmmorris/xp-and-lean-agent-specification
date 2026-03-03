---
name: tdd
description: Test-Driven Development workflow. Use for ALL code changes - features, bug fixes, refactoring. TDD is non-negotiable.
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
---

# Test-Driven Development

STARTER_CHARACTER = 🔴 for red (failing test), 🌱 for green (passing), 🌀 for refactor

TDD is the fundamental practice. Every line of production code must be written in response to a failing test.

**Related skills:**
- **For test patterns and anti-patterns**, load the `testing` skill
- **For refactoring guidance**, load the `refactoring` skill
- **For small increment planning**, load the `planning` skill

This skill focuses on the TDD workflow and discipline.

---

## Test Planning

Before writing any code, plan the full test list as `[TEST]` comment stubs. Convert them one at a time — never write ahead.

```
[TEST] Zero plus a number equals that number
[TEST] Add two positive numbers
[TEST] Add two negative numbers
[TEST] Division by zero returns an error
```

### ZOMBIES — Test Completeness Checklist

Walk through this before starting to catch gaps early. Source: [TDD Guided by ZOMBIES](https://blog.wingman-sw.com/tdd-guided-by-zombies) by James Grenning.

**ZOM axis** (simple → complex — work in this order):
- **Z** — Zero: initial/empty state, default values
- **O** — One: first item, first transition
- **M** — Many: multiple items, more complex scenarios

**BIE considerations** (apply at each ZOM level):
- **B** — Boundary: transitions between states in both directions (empty↔non-empty, not-full↔full)
- **I** — Interface: let tests reveal what methods, parameters, and return types are needed — don't design upfront
- **E** — Exceptions: error conditions; verify the object still works correctly after an error

**Key principles:**
- Test transitions, not just states — verify moving empty→non-empty and back
- Hard-coded return values are fine initially; tests will force you to generalise
- Get happy paths working first, then add error conditions

---

## RED-GREEN-REFACTOR Cycle

### RED: Write Failing Test First

Replace the next `[TEST]` comment with a real test. Before running:

**Predict the failure.** State explicitly what you expect to fail and why. If the test passes without any implementation, it is testing nothing.

The red phase has two steps:
1. **Compile failure** — the class or method doesn't exist yet. Add the minimal stub to make it compile.
2. **Assertion failure** — it compiles but returns the wrong value. This confirms the test is actually exercising something.

- Test describes desired behavior, not implementation
- Test should fail for the right reason

### GREEN: Minimum Code to Pass

**Predict whether the tests will pass.** Trace the logic before running.

- Write ONLY enough code to make the test pass
- Resist adding functionality not demanded by a test
- Commit immediately after green

**Simplification step** — after going green, examine every line you just added and ask: *"Does a failing test require this?"* If no test requires a line, delete it. If the behaviour is needed, add a `[TEST]` comment for it first.

This is the primary guard against speculative code.

### REFACTOR: Assess Improvements
- Assess AFTER every green (but only refactor if it adds value)
- Commit before refactoring
- All tests must pass after refactoring
- Consider whether a missing domain concept would make the code more expressive — new abstractions are fine as long as no new behaviour is introduced

---

## TDD Evidence in Commit History

### Default Expectation

Commit history should show clear RED → GREEN → REFACTOR progression.

**Ideal progression:**
```
commit abc123: test: add failing test for user authentication
commit def456: feat: implement user authentication to pass test
commit ghi789: refactor: extract validation logic for clarity
```

### Rare Exceptions

TDD evidence may not be linearly visible in commits in these cases:

**1. Multi-Session Work**
- Feature spans multiple development sessions
- Work done with TDD in each session
- Commits organized for PR clarity rather than strict TDD phases
- **Evidence**: Tests exist, all passing, implementation matches test requirements

**2. Context Continuation**
- Resuming from previous work
- Original RED phase done in previous session/commit
- Current work continues from that point
- **Evidence**: Reference to RED commit in PR description

**3. Refactoring Commits**
- Large refactors after GREEN
- Multiple small refactors combined into single commit
- All tests remained green throughout
- **Evidence**: Commit message notes "refactor only, no behavior change"

### Documenting Exceptions in PRs

When exception applies, document in PR description:

```markdown
## TDD Evidence

RED phase: commit c925187 (added failing tests for shopping cart)
GREEN phase: commits 5e0055b, 9a246d0 (implementation + bug fixes)
REFACTOR: commit 11dbd1a (test isolation improvements)

Test Evidence:
✅ 4/4 tests passing (7.7s with 4 workers)
```

**Important**: Exception is for EVIDENCE presentation, not TDD practice. TDD process must still be followed - these are cases where commit history doesn't perfectly reflect the process that was actually followed.

---

## Coverage Verification - CRITICAL

### NEVER Trust Coverage Claims Without Verification

**Always run coverage yourself before approving PRs.**

### Verification Process

**Before approving any PR claiming "100% coverage":**

1. Check out the branch
   ```bash
   git checkout feature-branch
   ```

2. Run coverage verification:
   ```bash
   cd packages/core
   pnpm test:coverage
   # OR
   pnpm exec vitest run --coverage
   ```

3. Verify ALL metrics hit 100%:
   - Lines: 100% ✅
   - Statements: 100% ✅
   - Branches: 100% ✅
   - Functions: 100% ✅

4. Check that tests are behavior-driven (not testing implementation details)

**For anti-patterns that create fake coverage (coverage theater)**, see the `testing` skill.

### Reading Coverage Output

Look for the "All files" line in coverage summary:

```
File           | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s
---------------|---------|----------|---------|---------|-------------------
All files      |     100 |      100 |     100 |     100 |
setup.ts       |     100 |      100 |     100 |     100 |
context.ts     |     100 |      100 |     100 |     100 |
endpoints.ts   |     100 |      100 |     100 |     100 |
```

✅ This is 100% coverage - all four metrics at 100%.

### Red Flags

Watch for these signs of incomplete coverage:

❌ **PR claims "100% coverage" but you haven't verified**
- Never trust claims without running coverage yourself

❌ **Coverage summary shows <100% on any metric**
```
All files      |   97.11 |    93.97 |   81.81 |   97.11 |
```
- This is NOT 100% coverage (Functions: 81.81%, Lines: 97.11%)

❌ **"Uncovered Line #s" column shows line numbers**
```
setup.ts       |   95.23 |      100 |      60 |   95.23 | 45-48, 52-55
```
- Lines 45-48 and 52-55 are not covered

❌ **Coverage gaps without explicit exception documentation**
- If coverage <100%, exception should be documented (see Exception Process below)

### When Coverage Drops, Ask

**"What business behavior am I not testing?"**

NOT "What line am I missing?"

Add tests for behavior, and coverage follows naturally.

---

## 100% Coverage Exception Process

### Default Rule: 100% Coverage Required

No exceptions without explicit approval and documentation.

### Requesting an Exception

If 100% coverage cannot be achieved:

**Step 1: Document in package README**

Explain:
- Current coverage metrics
- WHY 100% cannot be achieved in this package
- WHERE the missing coverage will come from (integration tests, E2E, etc.)

**Step 2: Get explicit approval**

From project maintainer or team lead

**Step 3: Document in AGENTS.md**

Under "Test Coverage: 100% Required" section, list the exception

**Example Exception:**

```markdown
## Current Exceptions

- **Next.js Adapter**: 86% function coverage
  - Documented in `/packages/nextjs-adapter/README.md`
  - Missing coverage from SSR functions (tested in E2E layer)
  - Approved: 2024-11-15
```

### Remember

The burden of proof is on the requester. 100% is the default expectation.

---

## Development Workflow

### Adding a New Feature

1. **Write failing test** - describe expected behavior
2. **Run test** - confirm it fails (`pnpm test:watch`)
3. **Implement minimum** - just enough to pass
4. **Run test** - confirm it passes
5. **Refactor if valuable** - improve code structure
6. **Commit** - with conventional commit message
7. **Ask: does this behavior need an e2e test?** - see below

### Workflow Example

```bash
# 1. Write failing test
it('should reject empty user names', () => {
  const result = createUser({ id: 'user-123', name: '' });
  expect(result.success).toBe(false);
}); # ❌ Test fails (no implementation)

# 2. Implement minimum code
if (user.name === '') {
  return { success: false, error: 'Name required' };
} # ✅ Test passes

# 3. Refactor if needed (extract validation, improve naming)

# 4. Commit
git add .
git commit -m "feat: reject empty user names"
```

### When Unit Tests Are Sufficient — Ask About E2E

When unit tests fully express a new behaviour, pause and ask:

> *"If all unit tests were deleted, would an e2e test catch a regression in this behaviour?"*

If yes, an e2e test is missing. Unit tests prove the logic is correct; e2e tests prove the behaviour reaches the user. Both are necessary — they are not substitutes for each other.

**The trigger is user-visible behaviour.** Ask this question whenever:
- A new response path is added (success message, error message, redirect)
- A new user interaction is handled (form submission, file upload, button click)
- An existing response is meaningfully changed (new field shown, wording changed)

**Do not duplicate edge cases in e2e.** Unit tests own the edge cases (invalid input, missing fields, mismatched names). E2E tests own at minimum one representative of each distinct outcome the user can see.

| Behaviour | Unit tests own | E2E tests own |
|-----------|---------------|---------------|
| Validation logic | All variants | One representative per error type |
| Success path | All variants | At least one confirming the full message reaches the user |
| Error path | All variants | At least one per distinct error the user sees |

---

## Commit Messages

Use conventional commits format:

```
feat: add user role-based permissions
fix: correct email validation regex
refactor: extract user validation logic
test: add edge cases for permission checks
docs: update architecture documentation
```

**Format:**
- `feat:` - New feature
- `fix:` - Bug fix
- `refactor:` - Code change that neither fixes bug nor adds feature
- `test:` - Adding or updating tests
- `docs:` - Documentation changes

---

## Pull Request Requirements

Before submitting PR:

- [ ] All tests must pass
- [ ] All linting and type checks must pass
- [ ] **Coverage verification REQUIRED** - claims must be verified before review/approval
- [ ] PRs focused on single feature or fix
- [ ] Include behavior description (not implementation details)

**Example PR Description:**

```markdown
## Summary

Adds support for user role-based permissions with configurable access levels.

## Behavior Changes

- Users can now have multiple roles with fine-grained permissions
- Permission check via `hasPermission(user, resource, action)`
- Default role assigned if not specified

## Test Evidence

✅ 42/42 tests passing
✅ 100% coverage verified (see coverage report)

## TDD Evidence

RED: commit 4a3b2c1 (failing tests for permission system)
GREEN: commit 5d4e3f2 (implementation)
REFACTOR: commit 6e5f4a3 (extract permission resolution logic)
```

---

## Refactoring Priority

After green, classify any issues:

| Priority | Action | Examples |
|----------|--------|----------|
| Critical | Fix now | Mutations, knowledge duplication, >3 levels nesting |
| High | This session | Magic numbers, unclear names, >30 line functions |
| Nice | Later | Minor naming, single-use helpers |
| Skip | Don't change | Already clean code |

For detailed refactoring methodology, load the `refactoring` skill.

---

## Final Evaluation

Once all `[TEST]` comments are implemented and passing:

1. **Check for gaps** — re-run ZOMBIES against the finished code. Are there cases the test list missed?
2. **Check for hardcoding** — is anything hard-coded that should be general? If so, add missing tests and implement properly.
3. **Check expressiveness** — is the code as clear as it could be? If not, go back to the refactoring phase.

---

## Anti-Patterns to Avoid

- ❌ Writing production code without failing test
- ❌ Testing implementation details (spies on internal methods)
- ❌ 1:1 mapping between test files and implementation files
- ❌ Using `let`/`beforeEach` for test data
- ❌ Trusting coverage claims without verification
- ❌ Mocking the function being tested
- ❌ Redefining schemas in test files
- ❌ Factories returning partial/incomplete objects
- ❌ Speculative code ("just in case" logic without tests)

**For detailed testing anti-patterns**, load the `testing` skill.

---

## Summary Checklist

Before marking work complete:

- [ ] Every production code line has a failing test that demanded it
- [ ] Commit history shows TDD evidence (or documented exception)
- [ ] All tests pass
- [ ] Coverage verified at 100% (or exception documented)
- [ ] Test factories used (no `let`/`beforeEach`)
- [ ] Tests verify behavior (not implementation details)
- [ ] Refactoring assessed and applied if valuable
- [ ] Conventional commit messages used
- [ ] Each user-visible outcome has at least one e2e test confirming it reaches the user
