---
name: polish
description: Review all current changes for quality, correctness, and consistency before committing or raising a PR. Find and fix issues directly.
---

# Polish Changes

Review all current changes for quality, correctness, and consistency before committing or raising a PR. Find and fix issues directly in the files, then produce a summary.

## Context

You are a senior engineer performing a final polish pass on in-progress work. Your goal is to catch problems that are easy to miss mid-flow: dead code, style inconsistencies, anti-patterns, and violations of codebase conventions.

Be surgical — fix what's wrong, leave what's right. Only touch code within the current diff; do not refactor unrelated files you happen to read.

Before starting, check for a `CLAUDE.md` or `AGENTS.md` in the current project directory and read it. Follow any project-specific conventions it defines.

## Process

### 1. Understand the Diff

```bash
git diff HEAD
git status
```

- All unstaged + staged changes
- New files not yet tracked

If the diff is empty and there are no untracked files, state that the working tree is clean and stop.

Read every changed and new file in full. Understand the intent before judging the implementation.

### 2. Run Verification (baseline)

Run the project's lint, type check, and test commands before making any edits so you know which failures pre-exist vs. which you introduce:

```bash
bundle exec rubocop --format simple
bundle exec rspec --format progress
bundle exec brakeman -q --no-pager
```

Note any pre-existing failures. Failures introduced by this pass must be resolved before it is complete. Pre-existing failures should be surfaced in the **Flagged** section — do not attempt to fix them.

### 3. Ruby & Rails Hygiene

Check each changed file for:

- **Unused variables / assignments** — remove any local variable assigned but never read
- **Dead code** — remove commented-out code, `puts`/`pp`/`debugger`/`binding.pry`/`byebug` statements; move any TODO comments to the Flagged section with a note on what they're blocking
- **N+1 queries** — check controller actions and scopes for missing `includes`/`preload`/`eager_load`; flag if a loop triggers individual queries
- **Missing strong parameters** — ensure controller actions use `params.require().permit()` properly; no mass-assignment of unpermitted attributes
- **Callback hygiene** — `before_action`, `after_action` in controllers should have proper `only:`/`except:` scoping; model callbacks should not have hidden side effects
- **Scope & query safety** — prefer scopes over raw SQL; use `where` with parameterised queries, never string interpolation
- **Validation completeness** — models should validate presence/format/uniqueness where the schema has `NOT NULL` or unique constraints
- **Route conventions** — RESTful routes preferred; no orphan routes pointing to non-existent controller actions

### 4. View & Stimulus Hygiene

Check changed view and JavaScript files for:

- **ERB safety** — use `<%=` (escaped) by default; `<%==` (raw) only when explicitly safe HTML is intended
- **Stimulus controller connections** — `data-controller` attributes match existing controllers in `app/javascript/controllers/`; `data-action` and `data-*-target` attributes reference valid methods/targets
- **Tailwind consistency** — follow existing patterns in the codebase (e.g., color palette, spacing conventions); no inline styles when Tailwind classes exist
- **Partial extraction** — if a view block is duplicated across files in the diff, extract to a partial
- **I18n** — user-facing strings should use `t()` helpers with keys in `config/locales/`; hardcoded Portuguese strings are acceptable only if the app is pt-BR only and follows existing patterns

### 5. Test Hygiene

Check changed spec files for:

- **Fabricator usage** — use `Fabricate(:model)` not `Model.create` directly; ensure fabricators exist in `spec/fabricators/`
- **Missing specs** — new public controller actions or model methods should have corresponding specs
- **Deterministic tests** — no reliance on database ordering without explicit `.order()`; no time-dependent tests without `travel_to`
- **Shoulda matchers** — prefer `should validate_presence_of(:field)` over hand-rolled validation tests where appropriate

### 6. Codebase Pattern Compliance

Check against patterns already established in this repo:

- **Controller structure** — follows the existing pattern of `before_action :set_resource` with private `set_*` and `*_params` methods
- **Model annotations** — if a model was changed and `annotaterb` annotations exist, note if they need updating
- **Concerns** — shared logic should be in concerns, not duplicated across controllers/models
- **Namespacing** — financial controllers under `Financial::`, obras controllers under `Obras::`; views mirror this structure
- **Service objects** — complex business logic should not live in controllers; suggest extraction if a controller action exceeds ~20 lines of business logic

### 7. Security Quick-Check

- **Brakeman** — if `bundle exec brakeman` reports new warnings on changed files, fix them
- **Authorization** — controller actions that modify data should have appropriate authorization checks
- **CSRF** — form submissions should go through Rails' built-in CSRF protection (no `skip_forgery_protection` unless justified)
- **File uploads** — ActiveStorage/Cloudinary uploads should validate content type and file size

### 8. Summary

After all fixes are applied, produce:

**Changes made**: A numbered list of every edit, with file path and one-line description.

**Flagged** (do not fix): Pre-existing issues, TODOs found in diff, or problems outside the scope of current changes.

**Verification result**: Output of lint/test/security commands after your edits — confirm no new failures were introduced.
