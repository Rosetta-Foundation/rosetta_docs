---
id: PRD-0026
title: Drop-Mode SDLC
status: Accepted # Draft | Proposed | Accepted | Shipped | Superseded | Deprecated
date: 2026-08-15
owner: Russ Watson
related_adrs: [ADR-0008]
related_specs: [] # filled when SPEC-PRD-0026-P1 lands on the engine repo
supersedes: null
---

# PRD-0026: Drop-Mode SDLC

> The automated SDLC's unit of ship is a **drop** (one worktree, one PR,
> machine gates merge); GitHub Issues are the inbox; Plan Mode is the plan
> review — not a second GitHub read of the same plan.

## 1. Overview & Goals

### 1.1 Purpose

PRD-0011 shipped a deterministic orchestration layer: well-defined scope,
failure rules, and machine language for reviews (HSR, tests, envelope). In
practice `run` made each spec **task** a worktree, a PR, a reviewer, CI, and
often a sandbox deploy. That grain costs a full agent bundle per micro-task
and halts the train when a diff grows ~150 lines or a dispatch exceeds a
flat token guess — after the operator has already planned the work.

Operator sessions in 2026-08 (intake → issues → one large drop PR →
watchers → sandbox verify → promote) already run the loop the operator
wants. This PRD makes that **drop** the default machine path. PRD-0011's
shipped phases stay historical fact; this PRD amends **grain**, **kickoff**,
and **budget**, not the evidence-gate idea.

### 1.2 Goals

- Make a **drop** the unit of ship: one worktree from a shared base, small
  commits, one PR that may be large, one reviewer pass, one CI, one sandbox
  deploy when the diff is deployable.
- Keep several drops in flight at once without clobbering — parallel
  worktrees, integration on the default branch, conflicts at merge.
- Use GitHub Issues as the durable inbox so intake is not lost. Write PRDs
  and ADRs when a contract must still bind in a year; they are planning
  artifacts, not a second kickoff ritual.
- Treat Cursor Plan Mode (accepted conversation) as the plan review before
  code. Do not require the operator to re-read that plan as a GitHub
  PRD/spec PR whose Approve is the only way to start the machine.
- For **direct** drops, enforce-mode merges on machine gates alone (house
  bar, CI, acceptance criteria, forbidden-surface envelope). Human Approve
  of the implementation PR is not required.
- Keep token `budgetK` as a signal; halt only at **3×** the envelope
  (raise later if needed). Stop using `maxDiffLines` as a hard gate.

### 1.3 Non-Goals

- Does not rewrite or unship PRD-0011 Phases 1–3; those specs remain Done.
- Does not auto-promote beyond the sandbox; production promote stays human.
- Does not generate PRDs or ADRs autonomously (PRD-0011 non-goal stands).
- Does not remove the reviewer agent, CI, or forbidden-surface envelope.
- Does not make style nits or a 150-line miss vs `maxDiffLines` a halt.
- Does not replace PRD-0020 (event daemon), PRD-0024 (planning-side skills),
  or PRD-0025 (operator-unstick) — those tracks stay their own.
- Does not require one worktree per spec **task**. Tasks are commits and
  acceptance-criteria checkboxes inside the drop.

### 1.4 Acceptance Criteria

- [ ] `sdlc-workflow` exposes a `drop` path whose input is one or more
      GitHub issues (a named drop / ship) and whose output is one PR from
      one worktree, with tasks recorded as commits + AC checkboxes.
- [ ] Two concurrent drops from the same default-branch tip use distinct
      worktrees and do not share a dirty working tree; the operator can
      open either worktree in an editor and read or edit in-flight code.
- [ ] A direct drop in enforce-mode merges when machine gates are green
      (reviewer house bar, CI, AC, forbidden surfaces) **without** a human
      `APPROVED` review on that implementation PR.
- [ ] Spend below **3×** `budgetK` never halts the drop; it appears on the
      run digest. Spend at or above 3× halts new agent dispatches and
      escalates (multiplier is config, default 3).
- [ ] Exceeding `maxDiffLines` does not fail the envelope gate when
      forbidden surfaces and allowed-path rules still hold; the digest
      notes the oversize.
- [x] After an accepted Plan Mode conversation, writing a PRD/ADR and
      starting the drop does not wait on a GitHub Approve of that docs PR
      as the start gun.
- [x] PRD-0011's records row and glossary **human gate** text point at
      this grain (drop = PR; spec-task ≠ PR) without changing shipped
      Phase 1–3 checkboxes.
- [x] `prd-lint` parses this PRD; `decompose` is not run until status is
      **Accepted**.

## 2. Users & Motivation

**Primary user: the operator (Watson).** The pain is paying a full
orchestration tax per micro-task and being asked to Approve a GitHub restatement
of a plan already accepted in chat — while still wanting to peek at code,
run several drops in parallel, and keep an inbox that does not forget.

**Secondary user: the engine and reviewer agents.** They already have
deterministic language (house bar, CI, AC, forbidden surfaces). This PRD
tells them to apply that language once per drop, not once per task, and to
merge on evidence for direct drops.

## 3. Approach

**Drop is the process.** Intake (transcript, Slack, prompt) is promoted to
GitHub Issues. A drop is a named set of those issues shipped together.

| Kind | When | Contract | Kickoff |
| --- | --- | --- | --- |
| Direct drop | Obvious, same-sitting, or a batch the operator already planned | Issue Done-when + optional PRD/ADR written as artifacts | Plan Mode accept in chat, or operator `drop` with issue refs |
| Needs a year-binding contract | Feature-sized or architectural | PRD and/or ADR **files** | Same — the files land; GitHub Approve of the docs PR is not the start gun |
| Bug with blast radius | Non-trivial defect | Bug spec (existing path) | Same drop grain: one PR, not one PR per task |

**Worktrees.** One worktree per in-flight drop, created from the same
up-to-date default-branch tip. The operator peeks in VS Code / Cursor.
Drops integrate on the default branch; merge conflicts at review are the
join. Parallel **tasks inside one drop** share that drop's worktree
(sequential commits) unless the operator opts into isolation.

**Gates (once per drop PR).**

| Gate | Hard? | On red |
| --- | --- | --- |
| Forbidden surfaces / allowed paths | Yes | Halt / remediate / escalate |
| Reviewer house bar (HSR, DRY, tests, inline docs) | Yes if item is `(mandatory)` | Remediate then unstick (PRD-0025); style nits do not halt |
| CI | Yes | Bounded fix cycle, then escalate |
| Acceptance criteria | Yes | Halt / remediate |
| `maxDiffLines` | No | Digest only |
| `budgetK` | Halt at **3×** only | Digest below 3×; escalate at 3× |

**Enforce merge.** For direct drops, green machine gates merge the
implementation PR as Addi (or the existing merge-on-gates path) **without**
waiting for a human Approve. Branch protection that today requires a human
review must be satisfied by a machine-gate path (App or Actions), not by
paging the operator to re-read the diff for style.

**Human remains.** Smoke the sandbox (and stakeholder verify where the
consumer uses it). Promote to production. Peek and edit in the worktree at
will. Authority-bound acts (live veto, regulated-data handling, flipping a
contract to Accepted when the operator has not accepted the plan) stay
human.

**PRD-0011 relationship.** Decompose may still write stories and tasks for
a large PRD. Those tasks are the drop's checklist, not each a PR. Existing
`run --max-parallel` worktree-per-task is opt-in, not default.

## 4. Data Contracts

```ts
/** Named ship: one or more inbox issues, one worktree, one PR. */
interface DropInput {
  dropId: string; // e.g. '2026-08-14' or 'acme-app#504'
  issues: string[]; // owner/repo#N
  baseRef: string; // default branch tip at arm time
  mode: 'direct' | 'bug-spec' | 'plan-artifact';
  /** When set, PRD/ADR files are written as artifacts, not kickoff gates. */
  artifactPrds?: string[]; // e.g. ['PRD-0026']
}

interface DropGrain {
  worktreePer: 'drop'; // not 'task'
  prPer: 'drop';
  tasksAre: 'commits-and-checkboxes';
  parallelDrops: true;
  parallelTasksInDrop: 'opt-in';
}

interface BudgetPolicy {
  budgetK: number; // envelope, same field as ADR-0008
  haltMultiplier: number; // default 3
  maxDiffLinesHardGate: false;
}

type DropMergeAuthority =
  | { kind: 'machine-gates'; appliesTo: 'direct' }
  | { kind: 'human-approve'; appliesTo: 'never-for-direct' };

interface DropEnvelope {
  allowedPaths: string[];
  forbiddenSurfaces: string[]; // existing .sdlc/surfaces.json labels only
  maxDiffLines: number; // advisory
  budgetK: number;
  haltAtBudgetK: number; // budgetK * haltMultiplier
}
```

## 5. Constraints & Dependencies

- Handler / Service / Repository + InversifyJS in `sdlc-workflow`;
  TypeScript strict; Conventional Commits + DCO.
- Envelope **forbidden-surface** labels must already exist on the target
  repo's `.sdlc/surfaces.json`. Do not invent labels at spec time. Engine
  repo today: `ci-config`, `personal-queue-schema`. Phase 3 may touch
  `.github/workflows/**` (`ci-config`) to let machine gates merge without
  a human Approve.
- Artifact hygiene: issue bodies, digests, and escalate text carry links
  and SHAs, never application data or production dumps.
- Addi authorship for engine PRs and issues (workspace GitHub App).
- Depends on PRD-0011 (gates, chronicle artifacts, no auto-promote),
  ADR-0008 (spec + envelope shape; `maxDiffLines` becomes advisory),
  PRD-0007 (queue / digest), PRD-0025 (unstick after remediable red gates).
- Soft-depends on consumer work-intake (issues as ledger). The engine
  speaks issue refs, not a consumer-only chat list.
- Platform boundary: editor peek is the operator's checkout of the drop
  worktree; the engine does not embed VS Code.

## 6. Risks & Open Questions

- Branch protection that requires a human `APPROVED` review will block
  enforce-merge until an App/Actions path is installed. Phase 3 must fail
  loud with that diagnosis, not spin.
- Operators may still want a proceed click on a scary drop. Mitigate with
  an opt-in `drop --require-approve` flag; default is machine-gates for
  `direct`.
- Huge PRs are harder for a reviewer agent. The house bar stays
  checklist-shaped; do not reintroduce per-task PRs to make review
  "easier."
- 3× may be too tight or too loose; keep it a single config default, not
  a per-task guess.
- Plan Mode accept is conversational. Persist a pointer (issue comment +
  Plan transcript ref) on the drop so kickoff is auditable without a
  GitHub spec-PR Approve.
- Should `bug-spec` / `plan-artifact` drops also machine-merge? Position
  for Phase 3: **direct** only; other modes keep today's merge-on-Approve
  until evidence says otherwise.

## 7. Rollout & Phases

1. **Phase 1 — Contract and envelope semantics:** Land this PRD and the
   PRD-0011 / glossary grain notes. Engine treats `maxDiffLines` as
   advisory and `budgetK` halt as `budgetK * haltMultiplier` (default 3).
   Existing `run` may still open per-task PRs until Phase 2; it must not
   hard-fail on line count or sub-3× budget. Tests + README.
2. **Phase 2 — Drop command and worktree-per-drop:** `sdlc-workflow drop`
   takes issue refs, creates one worktree from the shared base, implements
   as commits + AC checkboxes, opens one PR, runs gates once. Two drops
   from the same tip do not share a working tree. Parallel per-task
   worktrees become opt-in.
3. **Phase 3 — Enforce-merge for direct drops:** Green machine gates merge
   a `direct` drop PR without a human Approve. Fail loud if branch
   protection still requires a person. Docs PRD/spec Approves are not the
   start gun; Plan Mode accept + issue refs are.

## 8. Future Considerations

- Raise `haltMultiplier` from live digest evidence (5×, or per-repo).
- Extend machine-merge to `bug-spec` drops after a clean direct-drop track
  record.
- Distill a drop's Plan Mode transcript into the PRD file automatically
  (still not a kickoff gate).
- Consumer Slack sandbox-verify publish as an optional drop closeout hook
  (not a Rosetta-core gate).
