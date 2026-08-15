---
id: PRD-0025
title: Operator-First Escalation Unstick
status: Accepted # Draft | Proposed | Accepted | Shipped | Superseded | Deprecated
date: 2026-08-10
owner: Russ Watson
related_adrs: [] # e.g. [ADR-0002]
related_specs: [] # filled when SPEC-PRD-0025-P1 lands on the app repo default branch
supersedes: null
---

# PRD-0025: Operator-First Escalation Unstick

> After strict gate remediation fails, an operator-agent unsticks the SDLC run
> automatically; humans are paged only when the agent is stuck, and risky
> proceeds notify via non-blocking advisory issues.

## 1. Overview & Goals

### 1.1 Purpose

Today the engine already runs bounded **gate remediation** (same worktree,
verdict-scoped, envelope may only shrink) and then files a blocking
ACTION REQUIRED issue. In practice the operator pastes that issue into chat
with “handle this directly until resolved,” an agent with a broader mandate
unsticks the train, and the GitHub issue was mostly a pager with you as the
dispatcher. This PRD makes that middle tier first-class: after strict
remediation exhausts, dispatch an **operator-unstick** agent automatically;
keep the run moving; reserve human-blocking escalate for true stuckness; and
when the unstick agent chooses a **risky** path, proceed anyway but file a
non-blocking advisory issue so the operator can course-correct later.

### 1.2 Goals

- Remove the human-as-dispatcher loop for routine escalate unsticking while
  the operator is away or in another chat.
- Preserve a strict remediator that cannot widen envelopes or invent product
  policy — gates remain gates.
- Give the unstick agent an explicit operator mandate (rebase/integration tip,
  out-of-band merge + `record-merge`, resume) distinct from gate remediation.
- When the unstick agent proceeds under a risky assumption, the train keeps
  moving and an advisory GitHub issue records the decision for later review.
- Human-blocking escalate remains for budget exhaustion, abstention without a
  path, and authority-bound acts (Draft→Approved, live smoke/veto,
  regulated-data handling).

### 1.3 Non-Goals

- Not a replacement for gate remediation — that path stays first and strict.
- Does not let the unstick agent flip Draft→Approved, waive live smoke/veto,
  or widen `maxDiffLines` / `allowedPaths` in the Approved spec.
- Does not remove ACTION REQUIRED issues entirely — blocking issues still
  exist when the unstick agent cannot proceed; advisory issues are a separate
  class.
- Does not implement GitHub webhooks or cross-machine dispatch (stays on the
  local daemon / supervise path from PRD-0020).
- Does not teach the agent deploy/test mechanics beyond existing `.sdlc/`
  contracts and engine CLIs.

### 1.4 Acceptance Criteria

- [ ] After gate remediation exhausts for a remediable red gate (reviewer or
      envelope), the engine dispatches an operator-unstick agent turn without
      requiring a chat session, before or instead of a human-blocking
      ACTION REQUIRED issue.
- [ ] A successful unstick that clears blockers and records merge / resumes
      the run produces no human-blocking escalate issue for that wave.
- [ ] When the unstick agent labels a proceed as risky (or the engine
      classifies the chosen strategy as risky), the run continues and a
      non-blocking advisory GitHub issue is filed naming the decision,
      evidence links, and how to course-correct — closing that issue is not
      required for Continuity resume.
- [ ] When the unstick agent abstains or exhausts its budget without a
      cleared blocker, a human-blocking ACTION REQUIRED issue is filed (same
      durable wake / issue-state resume path as today).
- [ ] Gate remediation still cannot widen the envelope; tests prove an
      unstick prompt that attempts to raise `maxDiffLines` or edit mid-run
      `specs/**` for closeout is rejected or routed to abstain / advisory,
      not silent policy rewrite.
- [ ] `sdlc-workflow status` (or daemon status) distinguishes
      `unstick-in-flight`, `advisory-risky`, and `halted-escalated` so the
      operator can see which tier is active without reading chat.

## 2. Users & Motivation

**Primary user: the operator (Watson).** Today every escalate that an agent
can fix still costs a chat paste. The pain is babysitting latency, not lack
of capability. This PRD keeps Approve and smoke as human gates and removes
the “please unstick this” middleman role.

**Secondary user: the engine and headless agents.** Escalation wakes become
actionable by an operator-unstick headless turn (PRD-0020 Phase 3
headless dispatch) with a scoped prompt and transcript, so continuation does
not depend on an open editor session.

## 3. Approach

Three tiers after a red remediable gate (and analogous paths for
merge-blocked / push-failed exhaustion where an operator strategy differs
from “retry the same push”):

| Tier | Actor | Mandate | Blocks train? |
| ---- | ----- | ------- | ------------- |
| 1 — Gate remediate | Implementation agent | Fix inside verdict + envelope; trim only | No (bounded) |
| 2 — Operator unstick | Operator-agent | Unstick the run: alternate integration tip, OOB merge + `record-merge`, close obsolete blockers, resume | No while in flight; success clears |
| 3a — Advisory risky | Operator-agent + issue | Proceed with named risky assumption; file **non-blocking** advisory issue | **No** |
| 3b — Human escalate | Human | Stuck / no safe path / authority-bound | **Yes** |

**Unstick dispatch.** On remediation exhaustion, record
`state.unstickAttempts`, commit a wake (or in-process dispatch from
supervise) with HeadlessAction `agent-dispatch` and a fixed operator-unstick
prompt template: keep the train moving; prefer reversible evidence-backed
moves; classify each proceed as `safe` or `risky`; on `risky`, continue and
emit an advisory issue; on inability to proceed, abstain and human-escalate.

**Issue classes.**

- `ACTION REQUIRED: SDLC …` — blocking (existing). BlockerService /
  Continuity treat as unresolved blockers.
- `SDLC ADVISORY (risky unstick): …` — non-blocking. Must not appear in
  BlockerService resumable probes; optional `issue-state` watch for
  operator follow-up only.

**Budgets.** `gateFixAttempts` (existing) then `unstickAttempts` (new,
default 1–2). Exhaustion → human escalate. No unbounded unstick loops.

**Dependence on PRD-0020.** Phase 1 may dispatch unstick synchronously from
the supervise / run handler (same AgentRunnerRepository path as remediation)
so value lands before full daemon headless coverage. Phase 2 routes the
unstick wake through PRD-0020 Phase 3 HeadlessDispatchWakeAction when that
ships, so chat mirrors stay best-effort.

## 4. Data Contracts

```ts
/** Classification returned or inferred from an operator-unstick turn. */
type UnstickOutcome =
  | { kind: 'cleared'; summary: string; evidenceRefs?: string[] }
  | {
      kind: 'risky-cleared';
      summary: string;
      risk: string; // why this is risky
      evidenceRefs?: string[];
      advisoryIssueUrl: string;
    }
  | { kind: 'abstain'; reason: string }
  | { kind: 'failed'; reason: string };

interface UnstickAttemptRecord {
  taskId: string;
  attempt: number;
  startedAt: string;
  finishedAt?: string;
  outcome?: UnstickOutcome['kind'];
  transcriptRef?: string;
  advisoryIssueUrl?: string;
}

// RunState extensions (conceptual)
// unstickAttempts?: Record<taskId, number>
// unstickHistory?: UnstickAttemptRecord[]

/** Non-blocking advisory issue — must not use ACTION REQUIRED title prefix. */
interface AdvisoryRiskyIssue {
  title: string; // `SDLC ADVISORY (risky unstick): <runId> <taskId>`
  body: string; // decision, risk, evidence, course-correct hints
  labels?: string[]; // e.g. sdlc-advisory (optional)
}
```

## 5. Constraints & Dependencies

- HSR + InversifyJS; TypeScript strict; Conventional Commits + DCO.
- Workspace-agnostic: no hardcoded org/repo/login in engine code; operator
  login and activate script remain config (`--operator` / DaemonConfig).
- Artifact hygiene: advisory and escalate bodies carry links and SHAs,
  never application data or production dumps.
- Depends on existing gate remediation (`GateRemediationService`),
  `EscalationService`, BlockerService, and AgentRunnerRepository.
- Soft-depends on PRD-0020 Phase 3 headless wake dispatch for the preferred
  async path; Phase 1 of this PRD must work with in-process dispatch alone.
- Forbidden: mid-run `specs/**` status closeout as a silent unstick “fix”;
  that remains a separate docs/closeout path (PRD-0023).

## 6. Risks & Open Questions

- Agents may over-label `safe` to avoid advisory noise, or over-label
  `risky` and spam issues — mitigate with an explicit risk taxonomy in the
  prompt and tests for title/body shape.
- Unstick that force-pushes or rewrites history can surprise open reviews —
  prefer `--force-with-lease` / new tip branch patterns already used
  operationally; document in the prompt.
- Cost: unstick turns are larger than gate remediation; concurrency caps and
  per-task budgets are mandatory.
- Should merge-blocked / git-push-failed after a successful local commit skip
  straight to unstick (as seen when force-push is environment-blocked)?
  Recommend yes for Phase 1.
- Interaction with closeout “status unchanged / coverage incomplete” when
  phase gates were red but the task later merged OOB — advisory issues should
  link that class of risk for operator review.

## 7. Rollout & Phases

1. **Phase 1 — In-process operator unstick + advisory issues:** On
   remediation (and selected push/merge) exhaustion, dispatch operator-unstick
   via AgentRunnerRepository; record attempts; file advisory vs blocking
   issues per outcome; BlockerService ignores advisory titles; status
   surfaces the new states; tests + README/CHANGELOG.
2. **Phase 2 — Daemon headless unstick + wake wiring:** Register escalate /
   unstick wakes with HeadlessAction once PRD-0020 Phase 3 headless dispatch
   exists; chat notify remains a mirror; transcripts under a durable
   transcriptDir; optional digest line for open advisories.
3. **Phase 3 — Taxonomy hardening + operator UX:** Risk taxonomy labels,
   course-correct playbooks in advisory bodies, status/digest filters for
   `sdlc-advisory`, and tune default budgets from live evidence.

## 8. Future Considerations

- Auto-close advisory issues when a human comments `ack` or after a TTL with
  Chronicle note — never required for resume.
- Promote repeated advisory patterns into engine policy (so the next unstick
  is `safe` by construction).
- Queue-of-plans: an advisory on task N can attach a follow-up plan item
  without blocking task N+1.
