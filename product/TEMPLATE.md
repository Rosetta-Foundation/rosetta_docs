---
id: PRD-NNNN
title: <capability name>
status: Draft            # Draft | Proposed | Accepted | Superseded | Deprecated
date: YYYY-MM-DD
owner: <name>
related_adrs: []         # e.g. [ADR-0002]
related_specs: []        # e.g. [.sdd-phase1-spec.md] once implementation is sliced
supersedes: null
---

# PRD-NNNN: <capability name>

> One-sentence statement of what this capability is.

## 1. Overview & Goals

### 1.1 Purpose

<Why this exists — the problem it solves. 2–4 sentences.>

### 1.2 Goals

- <Goal — an outcome, not a task.>
- <Goal.>

### 1.3 Non-Goals

- <Explicitly out of scope. Non-goals prevent scope creep and are as important as goals.>

### 1.4 Acceptance Criteria

<Checkboxes so both a human and an agent can verify "done" against them.>

- [ ] <Observable, testable condition that must hold for this to ship.>
- [ ] <Condition.>

## 2. Users & Motivation

<Who this serves (engineers, managers, future agents) and the pain it removes.>

## 3. Approach

<The intended shape of the solution — enough to frame the phased specs, not the
full implementation. Prefer structure (lists, tables, typed contracts) over prose.>

## 4. Data Contracts

<Typed interfaces / schemas at the boundary, if any. Machine-legible.>

```ts
// example
```

## 5. Constraints & Dependencies

- <Constraint — privacy, platform, team standard (e.g. HSR pattern), etc.>
- <Dependency — another PRD, ADR, service, or external system.>

## 6. Risks & Open Questions

- <Risk or unresolved question for review.>

## 7. Rollout & Phases

<How this is sliced into shippable phases. Each phase becomes an SDD phase spec.>

1. **Phase 1** — <deliverable>
2. **Phase 2** — <deliverable>

## 8. Future Considerations

<What is deliberately deferred but anticipated.>
