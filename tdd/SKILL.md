---
name: tdd
description: Implement features from a plan using Red-Green-Refactor TDD. Write failing tests first, then minimal code to pass, then refactor.
argument-hint: "[plan item or feature description, e.g., 'item 1 from the plan' or 'add project archiving']"
allowed-tools: Read Glob Grep Edit Write Bash(pnpm *) Bash(npx *) Bash(git diff *) Bash(git status *) Bash(git log *) Bash(ls *) AskUserQuestion
---

# TDD — Test-Driven Development

Implement features by strictly following the Red-Green-Refactor cycle. Tests define the expected behavior _before_ any production code is written. Use this skill after `/build-plan` produces an approved plan, or when implementing any feature where correctness matters.

Inspired by the TDD-with-LLMs methodology: write expectations before implementations, use multiple test scenarios, and refactor with confidence knowing tests guard behavior.

## Arguments

$ARGUMENTS — What to implement. Accepts:

- A reference to a plan item from a previous `/build-plan` session (e.g., "item 1", "the auth feature from the plan")
- A feature description (e.g., "add bulk photo archive to projects")
- A file:line reference to existing code to extend with tests

If a plan exists in the conversation, use it as the source of truth for scope, files, and approach.

## Rules

- **RED before GREEN — always.** Never write production code before a failing test exists for the behavior being implemented. The only exception is frontend-only UI code that cannot be tested with Vitest (see Step 3 classification).
- **One cycle at a time.** Complete one Red-Green-Refactor cycle before starting the next. Do not batch multiple behaviors into a single cycle.
- **Minimal code to pass.** In the GREEN phase, write the simplest implementation that makes the failing test pass. Do not anticipate future tests or add logic beyond what the current test requires.
- **Tests define the contract.** Each test must describe the _expected_ behavior, not the current implementation. Write tests from the user's perspective: given these inputs, expect these outputs.
- **Multiple scenarios per behavior.** Never test a single happy path alone. Include at least: one happy path, one edge case, and one error/validation case per behavior.
- **Prioritize `_helpers/` functions.** Coverage targets are `convex/**/_helpers/**` (90% lines, 90% functions, 85% branches). These are pure TypeScript functions directly testable with Vitest. Convex query/mutation handlers require `convex-test` — install it when handler-level testing is needed.
- **Vitest globals are enabled.** `describe`, `it`, `expect` are available globally — do not import them from `vitest`.
- **Test files go inside `apps/backend/convex/`** — matching the pattern `convex/**/*.{test,spec}.{ts,tsx}`. Place test files next to the source they test (e.g., `convex/acl/_helpers/accessControl.test.ts`). Do NOT use `__tests__/` directories.
- **Follow project conventions.** Convex functions must have `args` and `returns` validators, use `.withIndex()` not `.filter()`, use `Id<"table">` not `string`. React components use MUI, pt-BR strings, and patterns from `docs/DEVELOPMENT_STANDARDS.md`.
- **This skill is for features, not bugs.** Use `/spec-fix` for bug fixes (one failing test → one fix). Use `/tdd` for implementing new features with multiple Red-Green-Refactor cycles.
- If $ARGUMENTS is empty or unclear, use AskUserQuestion to clarify what to implement.

## Steps

### 1. Understand the Scope

If a plan exists in the conversation from `/build-plan`, extract:

- Which items to implement (all, or the specific items requested)
- The files to change and create
- The tests listed in the plan
- The approach and constraints

If no plan exists, restate the feature in 1-2 sentences and identify:

- Which Convex functions, React components, or hooks are involved
- What the expected inputs and outputs are
- Which existing patterns to follow (use Glob/Grep to find similar features)

Read the relevant code:

- `apps/backend/convex/schema.ts` for data model
- Existing `*.test.ts` files in `apps/backend/convex/` for test patterns and helpers (if any exist — if this is the first test, see Step 4a for bootstrapping)
- Related Convex functions and `_helpers/` modules — these pure TS functions are the primary test targets
- `apps/backend/vitest.config.ts` for test configuration
- `docs/DEVELOPMENT_STANDARDS.md` for conventions

### 2. Plan the Test Scenarios

Before writing any code, list the test scenarios for the current behavior. Structure them as:

```
Behavior: <what the feature does>
  1. [happy] <description> — given X, expect Y
  2. [happy] <variant scenario> — given A, expect B
  3. [edge]  <boundary condition> — given edge input, expect handled correctly
  4. [error] <invalid input> — given bad input, expect ConvexError / validation failure
  5. [auth]  <unauthorized access> — given no auth / wrong role, expect rejection
```

Include auth and RBAC scenarios when the function checks permissions.

Output the scenario list briefly before proceeding — do not block for approval unless the scope seems wrong. If the plan already specifies tests, use those as the baseline and add missing edge/error cases.

### 3. Classify Each Behavior

Determine if the behavior is **test-able with Vitest**:

- **Yes — `_helpers/` functions** (pure TypeScript: access control checks, data transformations, business logic, validators): proceed with full Red-Green-Refactor cycle. These are the primary coverage targets.
- **Yes — Convex handlers** (queries, mutations, actions): requires `convex-test` package. If not installed, run `pnpm --filter @photoshot/backend add -D convex-test` first. Use `convexTest()` to set up a test environment with a real Convex backend.
- **Partially** (React hook logic, utility functions): write unit tests for the logic, skip visual rendering
- **No** (React UI rendering, MUI layout, routing, visual state): implement directly in the GREEN phase, provide manual verification checklist

### 4. RED — Write the Failing Test

For each test scenario (one at a time):

**a) Create or locate the test file:**

- Check `apps/backend/convex/` for an existing `*.test.ts` for the module. Add to it rather than creating a new file.
- If no test file exists, create one next to the source file inside `apps/backend/convex/` (e.g., `convex/acl/_helpers/accessControl.test.ts` for `accessControl.ts`).
- **First test in the project?** The project may have zero existing tests. If so, create the first test file following standard Vitest patterns — no special bootstrapping needed since `globals: true` is configured.

**b) Write the test:**

- Use `describe` blocks to group by behavior, `it` blocks for individual scenarios
- Name tests as expectations: `it("should reject when user has no access to project", ...)`
- Set up test data explicitly — no reliance on shared mutable state between tests
- Assert the _expected_ outcome, not the implementation details

**c) Run the test — confirm RED:**

```bash
pnpm --filter @photoshot/backend test <test_file>
```

The test MUST fail. If it passes, the test does not capture new behavior — revise it or the behavior is already implemented (skip to the next scenario).

If the test fails for the _wrong reason_ (syntax error, missing import, setup issue), fix the test infrastructure only — do not touch production code.

### 5. GREEN — Write Minimal Production Code

Write the simplest code that makes the failing test pass:

- Edit the Convex function, React component, or hook
- Add only what the test requires — no extra validation, no anticipated features, no "while I'm here" improvements
- Follow project conventions: `args`/`returns` validators, `.withIndex()`, `Id<"table">`, `ConvexError` for user-facing errors
- If schema changes are needed, update `apps/backend/convex/schema.ts` first

Run the test again — confirm GREEN:

```bash
pnpm --filter @photoshot/backend test <test_file>
```

If the test still fails, iterate on the production code. Do not modify the test to make it pass.

### 6. REFACTOR — Clean Up While Green

With the test passing, improve the code without changing behavior:

- Extract shared logic into helpers (in `_helpers/` directories for Convex)
- Remove duplication between the new code and existing code
- Ensure naming follows conventions
- Ensure the function signature is clean and consistent with neighboring functions

Run the full test file after refactoring:

```bash
pnpm --filter @photoshot/backend test <test_file>
```

All tests must still pass. If any test breaks during refactoring, undo the refactor and try a different approach.

### 7. Repeat

Go back to Step 4 for the next test scenario. Continue until all scenarios from Step 2 are covered.

When all scenarios for one behavior are done, move to the next behavior from the plan.

### 8. Full Verification

After all behaviors are implemented, run the complete test suite:

```bash
pnpm --filter @photoshot/backend test
```

Fix any regressions. If a pre-existing test breaks, investigate whether the new code changed shared behavior. If so, update the test to reflect the new correct behavior — do not silently skip it.

Then check coverage for `_helpers/` functions (thresholds: 90% lines, 90% functions, 85% branches, 90% statements):

```bash
pnpm --filter @photoshot/backend test:coverage
```

Report if any threshold is not met. If coverage drops, add more test scenarios.

For frontend-only changes, provide a manual verification checklist:

```
Verification:
1. Start dev server (pnpm dev)
2. Navigate to [specific URL]
3. [specific action to perform]
4. Expected: [what should happen]
5. Edge case: [another scenario to test manually]
```

### 9. Summary

**Feature**: One-sentence description of what was implemented.

**TDD Cycles completed**: Number of Red-Green-Refactor cycles executed.

**Tests written**:

| # | File | Test | Type |
|---|------|------|------|
| 1 | `path/to/test.test.ts` | `should do X when Y` | happy |
| 2 | `path/to/test.test.ts` | `should reject when Z` | error |

**Production code changed**:

| # | File | Change |
|---|------|--------|
| 1 | `path/to/file.ts` | Added function X |

**Manual verification** (frontend only): Checklist if applicable.

**Regressions**: None / list of pre-existing tests affected and how they were resolved.
