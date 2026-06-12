---
name: spec-fix
description: Write a failing spec for the described bug, then implement the minimal fix. For JS/view bugs that can't be spec'd, apply the fix with manual verification steps.
argument-hint: "[bug description, file:line, or URL]"
allowed-tools: Read Glob Grep Edit Write Bash(bundle exec *) Bash(git diff *) Bash(git status *)
---

# Spec → Fix

Write a failing spec that reproduces the bug, then implement the minimal fix to make it pass. For frontend bugs that can't be covered by RSpec, apply the fix directly with a manual verification checklist.

## Arguments

$ARGUMENTS — Bug description, URL, error message, or file:line reference. Can also reference a previous `/research` finding from the conversation.

## Rules

- **Spec first, fix second** — for backend bugs. Never touch application code before the spec exists and fails.
- **Fix first, verify manually** — for frontend bugs (Stimulus, inline JS, Turbo, CSS). These can't be caught by RSpec without system specs.
- **Minimal fix only.** Fix the bug — nothing else. No refactors, no cleanups, no improvements to surrounding code.
- **One bug, one spec file.** If the bug spans multiple areas, the spec still lives in one file covering the full scenario.
- **Follow existing spec patterns.** Use `Fabricate()` (not `Model.create`), `should` matchers, `sign_in` for auth. Use `Devise::Test::ControllerHelpers` in controller specs and `Devise::Test::IntegrationHelpers` in request specs.
- **Spec types:** `type: :controller` for controller actions, `type: :model` for model/validation, `type: :request` for routing/integration.
- Check `spec/support/` for shared helpers and contexts before writing setup code.
- Check `spec/fabricators/` for available fabricators before inventing attributes.
- If $ARGUMENTS is empty or unclear, use AskUserQuestion to clarify before writing anything.

## Steps

### 1. Understand the Bug

Restate the bug in one sentence. If $ARGUMENTS references a previous `/research` finding in the conversation, use that context directly — don't re-investigate from scratch.

If the bug is not clear enough, read the relevant code:

- Use Glob/Grep to find the affected files
- Read the controller action, model, view, or Stimulus controller involved
- Check `config/routes.rb` if routing is involved
- Check existing specs in `spec/` for the same area

### 2. Classify the Bug

Determine if the bug is **spec-testable**:

- **Yes** (model logic, controller action, API response, validation, query): → proceed to Step 3 (spec path)
- **No** (inline JS, Stimulus wiring, Turbo behavior, CSS, view rendering only): → skip to Step 5 (fix path) then Step 6 (manual verification)

### 3. Write the Spec (backend bugs only)

Create or edit the spec file:

- **Existing spec?** Check `spec/controllers/`, `spec/models/`, `spec/requests/` for a matching `*_spec.rb`. Add a new `describe`/`context` block rather than creating a new file.
- **No existing spec?** Create one: `spec/controllers/<name>_controller_spec.rb`, `spec/models/<name>_spec.rb`, or `spec/requests/<name>_spec.rb`.
- Use `let`/`let!` for setup, `before` for shared state (like `sign_in`).
- The spec MUST describe the **expected** behavior, not the current broken behavior.

### 4. Run the Spec (confirm red)

```bash
COVERAGE=false bundle exec rspec <spec_file>:<line> --format documentation
```

Confirm the new spec fails. If it passes, the spec doesn't capture the bug — revise it.

If `bundle exec` is not available, note it and proceed to the fix.

### 5. Implement the Fix

Apply the minimal change to make the spec pass (or fix the frontend bug):

- Read the application code involved
- Make the smallest edit that fixes the bug
- Prefer editing existing files over creating new ones
- Do NOT refactor, do NOT touch unrelated code

### 6. Verify

**For spec-testable bugs:**

```bash
COVERAGE=false bundle exec rspec <spec_file>:<line> --format documentation
```

Confirm the spec passes. If it still fails, iterate on the fix. Then run the full spec file:

```bash
COVERAGE=false bundle exec rspec <spec_file> --format documentation
```

**For frontend bugs (no spec):**

Provide a manual verification checklist:

```
Verification:
1. Start dev server (bin/dev)
2. Navigate to [specific URL]
3. [specific action to perform]
4. Expected: [what should happen]
5. Also test: hard-refresh the page and repeat
```

### 7. Summary

**Bug**: One-sentence description.

**Spec**: `path/to/spec_file.rb:line` — what it tests. _(or "N/A — frontend bug, manual verification above")_

**Fix**: `path/to/file.rb:line` — what changed and why.

**Verification**: Spec result or manual checklist.
