---
id: SPEC-BUG-<slug>-P1
prd: BUG-<slug> # synthetic id, not a real PRD file — free-text label only (see PRD-0011 §7 Phase 1 vs. this lightweight path)
phase: 1
status: Draft # Draft | Approved | Done | Superseded
date: YYYY-MM-DD
owner: <name>
envelope:
  allowedPaths: [] # globs the fix may touch, e.g. ["src/services/foo.ts", "src/**/__tests__/**"] — keep this tight, not repo-wide
  forbiddenSurfaces: [] # e.g. ["auth", "ci-config", "production-deploy"] — must already exist in the target repo's .sdlc/surfaces.json
  maxDiffLines: 200 # bugs are small by construction; keep this tight
  budgetK: 150
---

# SPEC-BUG-\<slug\>-P1: \<one-line bug summary\>

> Copy this template into the target repo as `specs/BUG-<slug>/phase-1-spec.md`
> (mirrors SPEC-TEMPLATE.md's convention, per ADR-0008). No PRD is written for
> this path — this is the lightweight bug entry point into the same
> spec-run-verify-merge machine (envelope, verification, reviewer, sandbox,
> provenance) that features go through via `/write-prd`. Use this only for
> non-trivial or blast-radius-sensitive bugs; a genuinely trivial, obviously
> safe one-liner doesn't need the machine — just fix it on a branch.

## Context

**Symptom:** \<what's observably wrong, from the user's or caller's perspective.\>

**Repro:** \<exact steps / inputs that trigger it. If there's a failing test,
name it; if not, describe how to reproduce manually.\>

**Root cause:** \<why it happens — the actual defect, not just the symptom.
If not yet known, say so explicitly and make T-01's first acceptance
criterion "root cause is identified and documented in the PR description"
before the fix criteria.\>

**Why now / blast radius:** \<why this needs the full machine rather than a
direct fix — e.g. touches a forbidden surface, non-obvious fix, or you want
it queued while you do something else.\>

## Task T-01: Fix \<symptom\>

- **Story:** S-01
- **Complexity:** S # bugs are almost always S; if this feels like M/L, reconsider whether it's really a bug or a small feature that deserves /write-prd instead
- **Depends on:** []

\<Engineering notes: the intended fix shape, and any constraint the
acceptance criteria don't capture (e.g. "don't change the public signature",
"preserve existing behavior for the non-buggy input range").\>

### Acceptance criteria

- [ ] test: \<a regression test that fails before the fix and passes after — this is the single most important criterion for a bug spec\>
- [ ] test: \<existing behavior for adjacent/non-buggy inputs is unchanged (no regression)\>
- [ ] agent: diff is confined to the fix and its test — no unrelated refactoring, no scope creep into other bugs noticed along the way
