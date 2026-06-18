---
name: kiro-spec
description: Produce Kiro-style spec-driven design artifacts (requirements, design, tasks) in docs/. Uses EARS acceptance criteria and codebase-anchored design. No implementation code.
argument-hint: "[feature name or slug] (e.g., 'InstaBook auto-booking for LTL', 'docs/insta-book from event storm')"
allowed-tools: Read Glob Grep Write Edit Bash(ls *) Bash(cat *) Bash(find *) Bash(mkdir *) Bash(git log *) Bash(git diff *) WebFetch WebSearch AskUserQuestion
---

# Kiro Spec (Spec-Driven Design)

Produce a **Kiro-style feature spec** in `docs/<feature-slug>/`: structured requirements (EARS), technical design, and an implementation task list. Anchors every claim in the Bloom codebase. **No application code** — only spec artifacts and optional `rules/` index.

**Workflow position:**

| Skill | Output | When |
| ----- | ------ | ---- |
| `/eventstorm-feature` | `docs/*-eventstorm.md` | Early discovery, open questions, informal flows |
| **`/kiro-spec`** | `docs/<slug>/requirements.md` + `design.md` + `tasks.md` | Requirements frozen enough to spec formally |
| `/build-plan` | Chat plan or execution | After spec approved; can use `tasks.md` as input |

## Arguments

$ARGUMENTS — Feature to spec. Accepts:

- A short title: `"InstaBook auto-booking for LTL"`
- A slug: `insta-book`
- A source doc: `"from docs/hb-auto-book-eventstorm.md"` or `"graduate docs/ops-add-project-and-need-eventstorm.md"`
- Stakeholder notes pasted inline (treat as source of truth for product intent)

If empty, use `AskUserQuestion` for feature name and whether an event-storm doc exists.

## Rules

- **NEVER write application code** (Convex, React, tests). Only spec markdown under `docs/<slug>/` and optional `rules/<slug>.md` index.
- **NEVER invent product decisions.** Unresolved items stay in **Open questions** — use `TBD` when the user defers; never `TODO` / `FIXME`.
- **Anchor design to code.** Entity names, mutations, routes, and gates must be verified via Read/Grep before appearing in design.md. Label guesses `(to verify)`.
- **EARS for acceptance criteria.** Every functional requirement uses WHEN/IF/WHILE/WHERE + **SHALL**. See [templates.md](templates.md).
- **Three artifacts minimum:** `requirements.md`, `design.md`, `tasks.md`, plus `README.md` index. Do not skip tasks.md.
- **Traceability:** Each task in `tasks.md` references requirement numbers (`_Requirements: R1, R3_`).
- **Reuse existing docs.** If `docs/*-eventstorm.md` exists, migrate decisions and flows into the Kiro spec; add a redirect stub in the old file (do not delete history-worthy content without user ask).
- **Scope discipline.** Explicit **In scope** / **Out of scope** in requirements. Non-goals must be testable exclusions.
- **Bloom conventions:** Convex backend, Next.js App Router, existing matchmaking/notification patterns — cite files, don't reinvent APIs.
- **Approval gate:** Present requirements for user review before writing design.md unless the user says "write full spec in one pass".

## Steps

### 1. Understand and choose slug

Restate the feature in one sentence. Derive **slug**: 2–5 kebab words, actor-action-object when possible (`insta-book`, `ops-add-project-and-need`).

Collect inputs:

- $ARGUMENTS
- Existing `docs/*-eventstorm.md` or product comments
- FigJam / Linear links (note in README, don't embed secrets)

If scope is unbounded ("redesign matchmaking"), ask the user to narrow before proceeding.

### 2. Discover domain anchors (read-only)

Same grounding as `/eventstorm-feature`:

- `docs/architecture/models.md`, `matchmaking-system.md`
- `packages/backend/convex/<domain>/`
- `apps/platform/src/app/(session)/**`
- Related `rules/*.md`

Produce a draft glossary (term → code path) for requirements.md.

**Timebox:** ~10 files, then ask if blocked.

### 3. Write `requirements.md`

Path: `docs/<slug>/requirements.md`

Structure (see [templates.md](templates.md#requirements)):

1. Document information table
2. Introduction (summary, business value, scope in/out)
3. Glossary
4. Numbered requirements **R1, R2, …** each with:
   - User story (As a … I want … so that …)
   - Acceptance criteria (EARS numbered list)
   - Additional details (priority, dependencies, assumptions)
5. Non-functional requirements (security, reliability, …)
6. Constraints and assumptions
7. Success criteria / definition of done
8. Open questions table
9. Requirements traceability matrix (optional, for multi-path features)

**EARS patterns:**

- `WHEN [event] THEN Bloom SHALL [response]`
- `IF [condition] THEN Bloom SHALL [behavior]`
- `WHILE [ongoing] Bloom SHALL [continuous behavior]`
- `WHERE [context] Bloom SHALL [contextual behavior]`

Use **Bloom** as the system name unless the feature is UI-only.

### 4. Review requirements with user

Summarize in chat:

- Requirement count and scope boundary
- Open questions count
- Any conflicts with existing code

If the user requests changes, `Edit` requirements.md before design.

### 5. Write `design.md`

Path: `docs/<slug>/design.md`

Structure (see [templates.md](templates.md#design)):

1. Overview + design goals + decision table
2. Architecture (mermaid: context, layers, sequences)
3. Flows / paths (all user-visible paths — e.g. Path 1 / Path 2)
4. Event storm or sequence diagrams (fenced ASCII or mermaid)
5. Data model (tables, new aggregates, consent fields)
6. Components (new queries/mutations/UI surfaces + **reuse** list with paths)
7. Security, error handling, testing strategy
8. Non-goals
9. References (requirements, rules, FigJam)

Map each design section to requirement IDs.

### 6. Write `tasks.md`

Path: `docs/<slug>/tasks.md`

Implementation plan as checkboxes, Kiro style:

```markdown
- [ ] 1. Title
  - Sub-step
  - _Requirements: R1, R2_
```

Group by: schema → backend → frontend → audit/tests → docs.

Order tasks by dependency. No file is too small to name if known from design.

### 7. Write `README.md` and wire references

Path: `docs/<slug>/README.md`

- One-paragraph summary
- Links to requirements, design, tasks
- Related artifacts (FigJam, prototype, event storm)
- Terminology mini-table

If superseding an event-storm doc, replace its body with a short redirect to `docs/<slug>/` (keep archived content in `<details>` or git history per user preference).

Optional: `rules/<slug>.md` — 30-line index pointing to spec (project convention).

### 8. Verify

Run this checklist before finishing:

- [ ] All requirements have EARS criteria
- [ ] Every open question is in requirements § Open questions (not silently decided)
- [ ] design.md references real file paths
- [ ] tasks.md links every task to at least one requirement
- [ ] No `TODO` in spec files
- [ ] README links resolve

### 9. Summary (chat output)

```markdown
## Spec created: <slug>

**Location:** `docs/<slug>/`

| Artifact | Path |
| -------- | ---- |
| Requirements | `docs/<slug>/requirements.md` |
| Design | `docs/<slug>/design.md` |
| Tasks | `docs/<slug>/tasks.md` |

**Requirements:** N (R1–Rn)
**Open questions:** M

**Next steps:**
- Resolve open questions in § Open questions
- `/build-plan docs/<slug>/tasks.md` or approve tasks for implementation
- `/skill-critique kiro-spec` (optional)
```

## Reference example

Canonical example in this repo: `docs/insta-book/` — InstaBook v2 (PR + AQ gates, Path 1 / Path 2, EARS requirements).

## Additional resources

- EARS templates and section skeletons: [templates.md](templates.md)
- Kiro methodology: https://kiro.dev/docs/specs/feature-specs/
