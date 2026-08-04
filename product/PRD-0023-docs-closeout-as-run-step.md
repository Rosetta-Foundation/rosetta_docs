---
id: PRD-0023
title: Docs Closeout as a Run Step
status: Draft # Draft | Proposed | Accepted | Shipped | Superseded | Deprecated
date: 2026-08-04
owner: Russ Watson
related_adrs: [ADR-0007, ADR-0008]
related_specs: []
supersedes: null
---

# PRD-0023: Docs Closeout as a Run Step

> The engine opens the spec-closeout docs PR itself — acceptance checkboxes
> flipped from verification verdicts, `status: Done`, and records tables
> updated — so documentation stays true as a byproduct of the run instead of
> a manual chore after it.

## 1. Overview & Goals

### 1.1 Purpose

The workspace principle is "documentation is a byproduct, context is the
product" — but stage 8 of the SDLC is entirely manual today. No code path
flips a delivered spec's acceptance checkboxes, sets `status: Done`, updates
the PRD/ADR records tables, or opens the closeout docs PR; operators do it
by hand (and the stacked-branch guidance for records tables exists precisely
because these manual PRs collide). Worse, the vacuum invites agents to
self-tick acceptance criteria inside product diffs (#40) — criteria state
must be engine-derived from verification verdicts, never agent-asserted.
This PRD makes closeout a final run step: derived, mechanical, and Addi-
authored, with the human's normal PR Approve as its only ceremony.

### 1.2 Goals

- Every enforce run that reaches "all tasks merged" produces a closeout
  docs PR with zero operator authoring: spec checkboxes flipped exactly
  where verification verdicts passed, `status: Done` when all criteria and
  phases hold, and the owning PRD's rollout/records entries updated.
- Checkbox state is derived solely from recorded verdicts — a criterion
  without a passing verdict stays unchecked and is listed in the PR body as
  an explicit remainder, so the closeout PR never overstates delivery.
- Closeout edits happen only through this path: product-task diffs
  continue to hard-breach on `specs/**`, and the closeout PR is the single
  writer for spec status.
- The definition of done becomes checkable: a phase is not reported
  complete in the digest until its closeout PR exists (merged or awaiting
  Approve).
- Runbook debt is surfaced, not silently skipped: when a spec declares an
  operational surface, the closeout PR includes the runbook stub or an
  explicit flagged gap.

### 1.3 Non-Goals

- Does not generate substantive prose: user-facing docs and runbook content
  beyond declared stubs remain authored work (agent- or human-), tracked as
  tasks — this PRD owns the mechanical truth of status artifacts.
- Does not merge its own PR: closeout PRs ride the existing
  merge-on-approve path like any Addi PR.
- Does not rewrite history for specs delivered before this ships.
- Does not replace Chronicle artifacts (ADR-0007) — closeout is the
  human-legible docs layer derived from the same verdicts.

### 1.4 Acceptance Criteria

- [ ] After an enforce run's final task merges, a closeout PR exists that
      flips exactly the spec checkboxes with passing verification verdicts,
      cites each verdict (task, gate, evidence link) in the PR body, and
      leaves failed/unverified criteria unchecked with a remainder list.
- [ ] A spec whose criteria and phases all pass gets `status: Done` in the
      same PR; a partially-delivered spec keeps its status with the
      remainder documented.
- [ ] When the spec belongs to a PRD, the PRD's rollout phase checklist and
      the product records table rows are updated in the same PR (stacked on
      any open records-table PR per the existing convention).
- [ ] The phase digest reports "complete" only after the closeout PR is
      open; the run's Chronicle artifacts link it.
- [ ] An agent-authored product diff that edits `specs/**` still
      hard-breaches the envelope gate (regression pinned; #40 stays
      closed by construction).
- [ ] A spec declaring an operational surface without a runbook produces a
      flagged gap in the closeout PR body rather than silence.
- [ ] Closeout is idempotent: re-running after interruption updates the
      same PR instead of opening duplicates.

## 2. Users & Motivation

**Primary user: the operator and reviewers.** Status tables that lie (specs
`Approved` forever, PRDs with stale phase checklists) make the portfolio
skills and human judgment work from fiction. Deriving them from verdicts
makes `sdlc-prd-progress` and the records tables trustworthy without anyone
remembering a chore.

**Secondary user: gate-policy calibration and audit.** A closeout PR that
cites verdict evidence per checkbox is the human-auditable face of the
Chronicle track record — the same evidence trail ADR-0007 requires, in the
place humans actually read.

## 3. Approach

A final engine step after the phase boundary (enforce mode), implemented as
a closeout service with the same HSR shape as digest/chronicle steps:

- **Verdict extraction** — read the run's recorded verification verdicts
  and merge records; map criteria → verdicts by the criterion identity the
  engine already tracks (never by re-parsing agent output).
- **Docs rendering** — apply checkbox flips and status transitions to the
  spec file; locate the owning PRD (spec front-matter `prd:` field) and
  update its rollout checklist and records-table row. Doc locations and
  table formats come from a small repo/workspace config (docs repo path,
  records file), never engine constants — consumers with different docs
  layouts declare their own.
- **PR mechanics** — a dedicated closeout branch and Addi-authored PR per
  run (idempotent upsert; re-runs update it), DCO-signed, conventional
  `docs(closeout):` messages; stacked on an open records-table PR when one
  exists, per the existing stacked-branch convention.
- **Digest integration** — the phase digest's completion claim requires the
  closeout PR reference; the veto path is unaffected (a vetoed phase's
  revert supersedes its closeout PR, which is closed with a note).

## 4. Data Contracts

```ts
// Closeout plan (engine-derived before any file edit)
interface CloseoutPlan {
  runId: string;
  specPath: string;
  checkboxFlips: Array<{
    criterion: string; // criterion identity from the spec
    taskId: string;
    verdictRef: string; // evidence link (chronicle artifact / PR)
  }>;
  statusTransition?: "Approved->Done"; // only when all criteria/phases hold
  remainders: string[]; // criteria left unchecked, with reasons
  prdUpdates?: {
    prdPath: string;
    phaseChecklist: Array<{ phase: number; done: boolean }>;
    recordsRow: string; // rendered per repo config
  };
  runbookGaps: string[]; // declared surfaces without runbooks
}

// Docs layout config (repo/workspace-owned, consumer-specific)
interface DocsCloseoutConfig {
  docsRepoPath?: string; // when PRD lives outside the target repo
  recordsFile: string; // e.g. "product/README.md"
  runbooksDir?: string;
}
```

## 5. Constraints & Dependencies

- Criteria state is engine-derived only — this PRD is the structural fix
  that keeps #40 closed; nothing in the closeout path accepts
  agent-asserted checkbox state.
- `specs/**` remains a hard-breach surface for product tasks; the closeout
  branch is the single sanctioned writer (enforced by the envelope gate's
  existing rule plus closeout running outside task envelopes).
- Platform boundary: docs layouts (records file, runbooks dir, docs repo)
  are consumer config; the engine ships extraction, rendering, and PR
  mechanics only.
- Depends on the verdict records that already exist (`sdlc.verdict.v1`);
  coordinates with PRD-0020 (closeout PR rides merge-on-approve and its
  watch) and SPEC-BUG-one-click-spec-approval (same Addi PR conventions).
- Addi authorship, DCO, conventional commits; no tool-attribution footers.
- HSR + InversifyJS, TypeScript strict.

## 6. Risks & Open Questions

- Cross-repo closeout (spec in the target repo, PRD in a docs repo) needs
  two PRs with one logical change; the plan links them and the digest
  reports both. Initial scope may restrict to same-repo PRD updates plus a
  flagged cross-repo remainder — decided in the phase spec.
- Records-table rendering is fragile to hand-edited table drift; the
  renderer must fail loud (flag, keep PR open with a note) rather than
  guess when the row shape doesn't match.
- A vetoed phase after closeout merged requires a revert of the closeout
  edits too; the veto path must treat the closeout PR as part of the
  phase's merge set.

## 7. Rollout & Phases

1. **Phase 1 — Same-repo closeout:** verdict extraction, checkbox flips,
   `status: Done`, idempotent Addi closeout PR, digest linkage; `specs/**`
   single-writer regression pinned.
2. **Phase 2 — PRD and records propagation:** owning-PRD rollout checklist
   - records-table updates via `DocsCloseoutConfig`, stacked-PR handling,
     cross-repo linking (or flagged remainder), veto-supersede handling.
3. **Phase 3 — Runbook surfacing:** declared operational surfaces produce
   runbook stubs or flagged gaps; closeout completeness feeds the digest's
   definition-of-done line.

## 8. Future Considerations

- Substantive doc generation (user-facing docs drafted from spec + diff)
  once the mechanical layer has track record.
- Closeout-derived release notes per phase.
- Consumer workbooks (e.g. Comita) subscribing to closeout events for their
  own status surfaces.
