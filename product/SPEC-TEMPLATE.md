---
id: SPEC-PRD-NNNN-Pn
prd: PRD-NNNN
phase: n
status: Draft # Draft | Approved | Done | Superseded
date: YYYY-MM-DD
owner: <name>
envelope:
  allowedPaths: [] # globs the run may modify, e.g. ["src/**"]
  forbiddenSurfaces: [] # e.g. ["migrations", "auth", "ci-config"]
  maxDiffLines: 1500
  budgetK: 200
---

# SPEC-PRD-NNNN-Pn: <phase deliverable>

> One-sentence statement of what this phase ships.
> Copy this template into the target repo as
> `specs/PRD-NNNN/phase-n-spec.md` (see ADR-0008).

## Context

<What this phase builds, from the parent PRD's rollout section. 2–4 sentences.
Link the PRD and any prior phase specs.>

## Task T-01: <title>

- **Story:** S-01
- **Complexity:** M # S | M | L
- **Depends on:** [] # task IDs

<Engineering notes: the intended shape, constraints, and anything an
implementation agent needs that the criteria don't capture.>

### Acceptance criteria

<Every criterion must carry a verification-tier tag. `test:` is asserted by a
scripted test in CI; `agent:` is verified by an agent using the running
sandbox interface; `manual:` requires a human and disables auto-advance for
the whole phase (ADR-0008).>

- [ ] test: <observable, deterministic condition>
- [ ] agent: <condition verified through the interface, evidence attached>

## Task T-02: <title>

- **Story:** S-02
- **Complexity:** S
- **Depends on:** [T-01]

<Notes.>

### Acceptance criteria

- [ ] test: <condition>
