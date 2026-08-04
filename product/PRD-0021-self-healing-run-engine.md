---
id: PRD-0021
title: Self-Healing Run Engine
status: Draft # Draft | Proposed | Accepted | Shipped | Superseded | Deprecated
date: 2026-08-04
owner: Russ Watson
related_adrs: [ADR-0007, ADR-0008]
related_specs: []
supersedes: null
---

# PRD-0021: Self-Healing Run Engine

> Every transient failure class in an SDLC run — not just CI — gets a
> bounded, evidence-recorded recovery path, so an overnight run survives
> flaky tests, stale verdicts, ecosystem outages, and dirty merge tips
> without a human noticing until the digest.

## 1. Overview & Goals

### 1.1 Purpose

Self-healing exists today only for the CI gate (a fix agent with failing
logs, three attempts). Every other failure class is terminal: an envelope or
reviewer breach has no retrigger path (one envelope breach halted a run
overnight), failed step digests are never retried (a salvage required
hand-patching `state.json`), gate verdicts cache indefinitely with no
invalidation on new evidence, a GitHub Actions billing outage had agents
"rewriting code to satisfy a CI that physically cannot start," and dirty-tip
merges have no fetch/rebase/retry. The operator postmortem line was
literally "there was no retrigger path for this class of failure." This PRD
gives the run engine a uniform recovery policy: transient failures retry
with bounds, stale verdicts invalidate on new evidence, external outages
pause rather than thrash, and only genuinely terminal states escalate.

### 1.2 Goals

- No single transient failure (flaky test, stale verdict, momentary API
  error, CI infrastructure outage, dirty merge tip) can halt a run
  overnight; each recovers within bounded attempts or escalates loudly with
  the recovery history attached.
- Gate verdicts are living state: a new push, a merged dependency, or a
  changed contract invalidates exactly the affected cached verdicts and
  re-gates — a red verdict never outlives the evidence that produced it.
- External-ecosystem failures (CI cannot start, GitHub API degraded) trip a
  circuit breaker: the run pauses, escalates once with the diagnosis, and
  resumes automatically when the ecosystem recovers — no agent dispatches
  against a broken environment.
- Run state mutations are crash-safe and single-writer: atomic writes and a
  run lock make the continuity layer's relaunch racing a manual resume
  impossible by construction.
- Every recovery is recorded as evidence (what failed, what was retried,
  what verdict changed) in the run artifacts and Chronicle, so gate policy
  can learn which failures are noise.

### 1.3 Non-Goals

- Does not weaken any gate: recovery retries the _evaluation_, never
  overrides a verdict; a genuine envelope breach or reviewer disagreement
  still escalates to the human.
- Does not add unbounded retry loops — every recovery path has an attempt
  cap and a terminal escalation.
- Does not cover watcher/wake delivery or supervisor resurrection
  (PRD-0020) or the silent-failure reporting seams
  (SPEC-BUG-fail-loud-run-lifecycle) — this PRD assumes failures are
  already loud and makes them recoverable.
- Does not introduce domain- or consumer-specific recovery rules; the
  policy is generic engine behavior.

### 1.4 Acceptance Criteria

- [ ] A planted flaky verification test (fails once, passes on rerun) does
      not halt the run: the verification gate re-executes within policy and
      the task advances, with both attempts recorded in the verdict
      evidence.
- [ ] After a task push that changes the diff, previously cached envelope
      and reviewer verdicts for that task are invalidated and re-gated
      automatically; a stale breach verdict can be shown (in tests) to never
      survive contradicting evidence.
- [ ] With CI check runs unable to start (simulated zero-check-runs /
      dispatch failure), the run pauses the affected tasks, files exactly
      one ecosystem escalation, dispatches no fix agents, and resumes
      unaided when checks become available.
- [ ] A merge rejected because the integration tip moved is recovered by
      fetch + rebase (or re-gate on the new tip) and retried within the
      attempt cap, without human action.
- [ ] A failed non-gate step (PR open, sandbox deploy, chronicle commit) is
      retried with backoff up to policy, and the run resumes from the step
      cache with no hand-edits to `state.json`.
- [ ] `state.json` writes are atomic (tmp + rename) and guarded by a run
      lock; a second writer attempt fails fast with a clear error instead
      of clobbering state.
- [ ] Dispatched implementation/fix agents run with a sanitized
      environment; a nested `CURSOR_AGENT=1` (or equivalent) can no longer
      make dispatched agents silently no-op.
- [ ] Every automated recovery emits a structured record (failure class,
      attempts, outcome) into the run's Chronicle artifacts.

## 2. Users & Motivation

**Primary user: the operator.** The promise of the automated SDLC is
"advance on evidence while I sleep." Today one transient failure forfeits
the night: the 66281ecb and 97576118 sessions each lost hours to failures
the machine could have retried (a flaky frontend suite, a stale breach
verdict that outlived its evidence by minutes, a billing outage that burned
agent turns). Recovery-with-bounds converts those into digest footnotes.

**Secondary user: gate-policy calibration.** Recovery records tell the
Chronicle which gate failures are environmental noise versus real defects —
the input PRD-0011 §8 needs for progressive trust.

## 3. Approach

A uniform recovery policy layer inside the run engine (no new process, no
new contract surface), applied at three levels:

- **Step level** — a failed step (today: left unrecorded, retried only on
  manual re-invocation) gets an explicit retry record with capped attempts
  and backoff; the step cache distinguishes "never ran," "failed
  (retryable, N attempts left)," and "failed terminal."
- **Verdict level** — cached gate verdicts carry the digest of the evidence
  they judged (diff SHA, contract blob, tip SHA); the engine invalidates
  verdicts whose evidence digest no longer matches reality before every
  gate evaluation, extending the existing gate auto-recover work
  (fork-landed) to all four gates uniformly.
- **Environment level** — circuit breakers for external systems: CI
  (zero check runs ≠ terminal; StatusContext-only PRs supported; repeated
  cannot-start trips the breaker), GitHub API (backoff + resume), and the
  merge tip (fetch/rebase/retry). A tripped breaker pauses affected tasks,
  escalates once, and probes for recovery on the supervise cadence.

Supporting hardening in the same scope: atomic `state.json` writes with a
run lock (single-writer), and environment sanitation for dispatched agents.
The 1,471-line `run.handler.ts` is carved opportunistically as each gate's
recovery is touched — extraction is in scope only where it serves the
recovery work, not as a big-bang refactor.

## 4. Data Contracts

```ts
// Recovery policy (engine defaults; overridable per run invocation)
interface RecoveryPolicy {
  stepMaxAttempts: number; // default 3
  stepBackoffSeconds: number[]; // e.g. [30, 120, 600]
  verificationReruns: number; // flaky-test reruns, default 1
  ciFixAttempts: number; // existing, default 3
  breakerProbeSeconds: number; // ecosystem recovery probe cadence
}

// Step retry record (persisted in state.json per step key)
interface StepAttempt {
  attempt: number;
  failedAt: string;
  failureClass:
    | "transient-exec" // nonzero exit, retryable
    | "external-outage" // breaker-owned
    | "stale-evidence" // verdict invalidated
    | "terminal"; // escalated
  detail: string;
}

// Verdict evidence binding (what a cached verdict judged)
interface VerdictEvidence {
  diffSha: string; // task head judged
  baseSha: string; // integration tip judged against
  contractDigest?: string; // .sdlc blob digest (envelope/verification)
}

// Circuit breaker state (per external system, per run)
interface BreakerState {
  system: "ci" | "github-api" | "merge-tip";
  trippedAt?: string;
  lastProbeAt?: string;
  escalated: boolean; // exactly-once escalation per trip
}

// Recovery record (Chronicle artifact, sdlc.recovery.v1)
interface RecoveryRecord {
  runId: string;
  taskId?: string;
  failureClass: string;
  attempts: StepAttempt[];
  outcome: "recovered" | "escalated";
}
```

## 5. Constraints & Dependencies

- HSR + InversifyJS, TypeScript strict — recovery services follow the same
  architecture bar as the rest of the engine.
- Never overrides a gate verdict: recovery re-evaluates with fresh
  evidence; verdict semantics (pass/fail/human-required) are unchanged.
- Builds on the digest-keyed step cache (kept per the gap-analysis
  fix-versus-rebuild call) — recovery extends the cache's state model, it
  does not replace it.
- Sequenced after SPEC-BUG-fail-loud-run-lifecycle (failures must be
  visible before they are recoverable) and alongside PRD-0020 (breaker
  escalations deliver through the daemon's wake path when present).
- Chronicle artifact addition (`sdlc.recovery.v1`) follows ADR-0007
  provenance conventions.
- Domain-agnostic: no consumer-specific retry rules; consumers influence
  behavior only through their `.sdlc/` contracts (e.g. a verification
  command that is itself retry-aware).

## 6. Risks & Open Questions

- Flaky-test reruns can mask real intermittent defects; mitigation: every
  rerun is recorded as evidence, and repeated flake recoveries on the same
  criterion surface in the digest for human attention.
- Verdict invalidation must be surgical — over-invalidation re-runs
  expensive reviewer/verifier agents; the evidence-digest binding must be
  exact (the #48 lesson: a changed digest once re-opened merged tasks).
- Run locks must not deadlock legitimate takeover (a dead supervisor's
  lock); lock records carry the holder PID and staleness rules.
- Breaker false positives (declaring CI down when one workflow is
  misconfigured) — the probe distinguishes "cannot start anywhere" from
  "this PR's checks failed to start."

## 7. Rollout & Phases

1. **Phase 1 — Crash-safe state + step retry policy:** atomic writes, run
   lock, step attempt records with capped backoff retries, environment
   sanitation for dispatched agents; salvage-by-hand-patching becomes
   unnecessary.
2. **Phase 2 — Verdict evidence binding + uniform gate retrigger:** cached
   verdicts bound to evidence digests; automatic invalidation and re-gate
   for envelope, reviewer, and verification on new pushes / merged
   dependencies / contract changes; flaky-verification rerun policy.
3. **Phase 3 — Circuit breakers + recovery records:** CI/GitHub/merge-tip
   breakers with pause-escalate-probe-resume semantics; `sdlc.recovery.v1`
   Chronicle artifacts; digest surfacing of repeated flake recoveries.

## 8. Future Considerations

- Chronicle-driven adaptive policy: attempt caps and rerun budgets tuned
  per repo / per gate from the recovery track record (PRD-0011 §8
  progressive trust).
- Cross-run breaker state (an ecosystem outage observed by one run
  pre-trips the breaker for siblings).
- Recovery-aware ETA in `status` (attempts remaining as part of the
  in-flight answer).
