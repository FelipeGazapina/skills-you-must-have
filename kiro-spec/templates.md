# Kiro Spec — Templates

Copy sections into `docs/<slug>/` artifacts. Replace bracketed placeholders.

---

## requirements

```markdown
# [Feature Name] — Requirements

## Document information

| Field | Value |
| ----- | ----- |
| **Feature name** | [Name] |
| **Version** | 1.0 |
| **Date** | [YYYY-MM-DD] |
| **Status** | Draft / Approved for implementation |
| **Stakeholders** | [Roles] |
| **Related** | [design.md](./design.md) |

---

## Introduction

[2–3 paragraphs: problem, solution summary, how it fits Bloom]

### Feature summary

[One sentence]

### Business value

- [Outcome 1]
- [Outcome 2]

### Scope

**In scope (v1)**

- [Item]

**Out of scope (v1)**

- [Item]

---

## Glossary

| Term | Definition |
| ---- | ---------- |
| [Term] | [Definition + code path if domain-specific] |

---

## Requirements

### Requirement 1: [Title]

**User story:** As **[role]**, I want **[capability]**, so that **[benefit]**.

#### Acceptance criteria

1. WHEN [event] THEN Bloom SHALL [response]
2. IF [condition] THEN Bloom SHALL [behavior]
3. WHILE [ongoing condition] Bloom SHALL [continuous behavior]
4. WHERE [context] Bloom SHALL [contextual behavior]

#### Additional details

- **Priority:** High / Medium / Low
- **Dependencies:** [R# or external system]
- **Assumptions:** [List]

### Requirement 2: [Title]

[Repeat structure]

---

## Non-functional requirements

### Security and authorization

1. WHEN [access] THEN Bloom SHALL [enforce pattern e.g. withProjectAlterAccess]

### Reliability

1. WHEN [operation] THEN Bloom SHALL [idempotency / recovery]

---

## Constraints and assumptions

### Technical constraints

- [Convex, reuse existing mutations, etc.]

### Business constraints

- [Rollout, actors, Ops-only, etc.]

### Assumptions

- [List]

---

## Success criteria

### Definition of done

- [ ] [Measurable outcome]
- [ ] [Measurable outcome]

---

## Open questions

| # | Question | Impact |
| - | -------- | ------ |
| 1 | [Question] | [Why it matters] |

---

## Requirements traceability

| Req | [Flow A] | [Flow B] | Notes |
| --- | -------- | -------- | ----- |
| R1 | ✓ | — | |
```

### EARS quick reference

| Pattern | Template | Example |
| ------- | -------- | ------- |
| Event | WHEN … THEN … SHALL … | WHEN the HB declines Path 2 THEN Bloom SHALL NOT create a relationship row |
| Condition | IF … THEN … SHALL … | IF no priceResolver THEN Bloom SHALL NOT offer InstaBook |
| Continuous | WHILE … SHALL … | WHILE InstaBook is active THEN Bloom SHALL write audit events |
| Context | WHERE … SHALL … | WHERE the user is Ops THEN Bloom SHALL allow relationship CRUD |

---

## design

```markdown
# [Feature Name] — Design

## Document information

| Field | Value |
| ----- | ----- |
| **Version** | 1.0 |
| **Related** | [requirements.md](./requirements.md) |

---

## Overview

[How design addresses requirements]

### Design goals

1. [Goal]
2. [Goal]

### Key design decisions

| Decision | Choice | Rationale |
| -------- | ------ | --------- |
| [Topic] | [Choice] | [Why] |

---

## Architecture

### System context

\`\`\`mermaid
flowchart TB
  [Diagram]
\`\`\`

### [Layer / gate / component diagram]

[Explain]

---

## Flows

### [Path or flow name]

**Trigger:** [When]

**Steps:** [Numbered]

\`\`\`mermaid
sequenceDiagram
  [Sequence]
\`\`\`

---

## Event storm (optional)

\`\`\`
👤 Actor
    ▼
🟧 Command: [name]
    ▼
🟨 Event: [name]
\`\`\`

---

## Data model

### New: [table name]

| Field | Type | Notes |
| ----- | ---- | ----- |

**Invariant:** [Rule]

### Changes to existing aggregates

- [Field on need/project/service]

---

## Components

### Backend (new)

| Component | Type | Responsibility |
| --------- | ---- | -------------- |

### Frontend (new)

| Surface | Flow | Notes |

### Reuse (existing)

| Area | Path |
| ---- | ---- |

---

## Security

- [Auth pattern]

## Error handling

| Condition | Behavior |
| --------- | -------- |

## Testing strategy

### Happy paths

1. [Scenario]

### Corner cases

1. [Scenario]

---

## Non-goals

- [Item]

## References

- [requirements.md](./requirements.md)
- [rules/...](../../rules/...)
```

---

## tasks

```markdown
# [Feature Name] — Implementation Plan

> Trace to [requirements.md](./requirements.md) and [design.md](./design.md).

- [ ] 1. Schema and types
  - Add `tableName` to `packages/backend/convex/schema` or `*_tables.ts`
  - Indexes: [list]
  - _Requirements: R1_

- [ ] 2. Backend — eligibility
  - `evaluateX` query in `packages/backend/convex/...`
  - Unit tests in `*.test.ts`
  - _Requirements: R2_

- [ ] 3. Backend — mutations
  - `recordY` mutation with validators + auth
  - _Requirements: R3, R4_

- [ ] 4. Frontend — [surface]
  - Component at `apps/platform/src/...`
  - Wire into [existing panel/route]
  - _Requirements: R4_

- [ ] 5. Audit and observability
  - Structured logs / events
  - _Requirements: R6_

- [ ] 6. Documentation
  - Update `rules/<slug>.md` if shipped
  - _Requirements: —_
```

---

## readme

```markdown
# [Feature Name] — Spec

[Kiro-style spec index]

| Artifact | Purpose |
| -------- | ------- |
| [requirements.md](./requirements.md) | EARS requirements |
| [design.md](./design.md) | Architecture and flows |
| [tasks.md](./tasks.md) | Implementation checklist |

## Related

- Event storm: [link or path]
- FigJam: [url]
- Prototype: [path]

## Terminology

| Term | Meaning |
| ---- | ------- |
```
