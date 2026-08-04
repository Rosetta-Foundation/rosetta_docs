---
id: PRD-0022
title: Delivery Truth & Run Observability
status: Accepted # Draft | Proposed | Accepted | Shipped | Superseded | Deprecated
date: 2026-08-04
owner: Russ Watson
related_adrs: [ADR-0007, ADR-0008]
related_specs: [SPEC-PRD-0011-P4]
supersedes: null
---

# PRD-0022: Delivery Truth & Run Observability

> Close the trust gap between "gates green" and "feature actually reachable
> in the sandbox," and give a running SDLC workflow an honest, always-current
> answer to "what is it doing and how much longer?"

## 1. Overview & Goals

### 1.1 Purpose

Deceptive signals cost more trust than loud failures. Live runs produced
green verdicts for undelivered features (sandbox health passed on the right
SHA while the feature's `/version.json` 404'd on the CDN and a deploy `prune`
silently deleted `robots.txt`), a "green" deploy workflow that had executed
only its change-detection job and shipped nothing, and a "red" one that had
actually succeeded. Meanwhile the run itself is opaque exactly when it
matters: heartbeats repeat the last step label through 15-minute CI polls and
sandbox waits, `agentAlive` greps for any agent on the machine, agent thrash
has no convergence signal, and "how much longer?" has no answer. The same
period wasted deploys: the build verifier waits for the PR-head deploy, the
merge redeploys the same content, and a phase-boundary deploy raced the push
deploy into a CloudFormation conflict. This PRD makes delivery verification
content-level, deploys deduplicated, and run progress honestly observable —
finishing PRD-0011 Phase 4 (path-aware sandbox deploy) as part of the same
delivery-cost work.

### 1.2 Goals

- A phase is "delivered" only when content-level assertions pass against the
  live sandbox: the shipped artifact is fetched and matched, negative
  assertions guard collateral damage, and workflow status alone is never
  accepted as delivery evidence.
- Deploy work is spent once per content SHA: the merge reuses the PR-head
  deploy when SHAs match, the racing phase-boundary deploy is eliminated,
  and docs-only diffs fast-pass (PRD-0011 Phase 4 landed and validated).
- The heartbeat tells the truth through every step: CI polling, sandbox
  deploy/health waits, verification, and chronicle/digest steps update step
  context, and `agentAlive` refers to this run's agent, not any agent on the
  machine.
- Thrash is detected: an implementation or fix agent showing no convergence
  (no new commits, repeating diffs, or wall-clock past its complexity
  budget) surfaces a convergence warning instead of burning tokens silently.
- `status` answers the operator's actual questions: what is each task doing
  now, what gates remain, what recovery attempts are pending, and an
  evidence-based ETA band from Chronicle track records of comparable tasks.

### 1.3 Non-Goals

- The engine does not learn deploy or content mechanics — what to curl and
  what must never disappear are repo-owned assertions in `.sdlc/` scripts;
  the engine only defines the contract hook and passes SHAs.
- Not an APM or general observability platform — scope is the run's own
  progress surface and the delivery verdict, per ADR-0004's
  deliberate-observability posture (production alarm design stays with the
  consumer's go-live checklist).
- No cross-surface end-to-end auth verification (the accounts/admit
  `redirect_uri` class) — that regression floor belongs to the consumer's
  scripted E2E suite, tracked separately.
- Does not change gate semantics or add human touchpoints.

### 1.4 Acceptance Criteria

- [ ] The sandbox delivery verdict for a task/phase includes content-level
      evidence: at least one fetched-artifact assertion tied to the deployed
      SHA, with the raw fetch output attached; a workflow-status-only
      "healthy" is impossible to produce through the contract.
- [ ] A repo-declared negative assertion (e.g. a must-exist path like
      `robots.txt`) failing after deploy turns the delivery verdict red even
      when the app health endpoint is green.
- [ ] A deploy workflow run that completed without executing its ship jobs
      (change-detection-only) is classified as "did not deploy," not as
      delivery success.
- [ ] When the merged SHA's content equals an already-deployed PR-head SHA,
      the merge path records reuse instead of dispatching a second deploy;
      the phase boundary never dispatches a deploy that races a push deploy.
- [ ] `heartbeat.jsonl` step context changes across implementation → PR →
      gates → CI poll → sandbox deploy → verification → chronicle, with no
      step longer than one interval reporting a stale label; `agentAlive`
      is true only for the run's own dispatched agent process tree.
- [ ] A planted non-converging agent (loops without committing) produces a
      thrash warning in the monitor log and digest within its complexity
      budget, without killing legitimately slow work.
- [ ] `status --run-id` reports per-task current step, remaining gates,
      pending recovery attempts, and an ETA band with its evidence basis;
      the same summary is available in the phase digest.
- [ ] PRD-0011 Phase 4 acceptance holds end-to-end in a live run: a
      docs-only task fast-passes the sandbox contract in seconds while a
      deployable task still ships and health-checks the real SHA.

## 2. Users & Motivation

**Primary user: the operator.** The sandbox smoke is one of exactly two
human touchpoints — it must be spent on judgment, not on discovering that
"green" meant "nothing shipped." The `/version.json`, `robots.txt`, and
502-behind-a-green-alarm incidents each converted the smoke into unpaid
debugging. Honest ETA and step context replace the "You still good bro?"
check-ins that the heartbeat was supposed to eliminate.

**Secondary user: the machinery itself.** PRD-0020's daemon and the
PRD-0021 breakers need truthful step context and delivery classification to
decide when to wake, retry, or escalate.

## 3. Approach

Three strands, split cleanly across the platform boundary:

- **Delivery truth (contract hook upstream, assertions consumer-side).**
  Extend the `.sdlc/` verification/environments contract schema with a
  content-assertion hook: a repo-owned script the engine invokes after
  deploy with `SDLC_SANDBOX_SHA` / `SDLC_SANDBOX_BASE_SHA`, whose structured
  output (per-assertion pass/fail + raw evidence) feeds the sandbox verdict.
  The engine also classifies deploy workflow runs (shipped / fast-passed /
  did-not-deploy) instead of trusting conclusion=success. First consumer:
  `comita_admissions` assertions (version artifact fetch, must-exist paths,
  log negative-assertions) in its `.sdlc/` scripts — landed as a companion
  consumer PR, never in engine code.
- **Deploy dedup (engine).** The sandbox step records deployed content SHAs;
  merge and phase-boundary paths consult the record and reuse instead of
  re-dispatching; the redundant phase-boundary dispatch is removed. Finish
  and live-validate SPEC-PRD-0011-P4 (skip / thin-dispatch) as part of this
  strand.
- **Run observability (engine).** Step-context coverage for the silent
  steps; per-run agent identity (dispatch records the child process
  group; `agentAlive` checks that tree); a convergence monitor over agent
  activity (commit cadence, diff delta, budget clock); `status` and digest
  enriched with per-task step, remaining gates, recovery state (PRD-0021),
  and an ETA band computed from Chronicle `sdlc.merge.v1` history for
  comparable complexity tasks.

## 4. Data Contracts

```ts
// Content assertion result (repo script output, engine-validated)
interface ContentAssertionReport {
  sha: string; // SDLC_SANDBOX_SHA asserted against
  assertions: Array<{
    name: string; // e.g. "version-artifact", "robots-txt-present"
    kind: "fetch-match" | "must-exist" | "must-not-exist" | "log-absent";
    pass: boolean;
    evidence: string; // raw curl/zip/log excerpt
  }>;
}

// Deploy classification (engine-derived, per dispatch or reuse)
interface DeployRecord {
  contentSha: string;
  outcome: "shipped" | "fast-passed" | "reused" | "did-not-deploy";
  workflowRunId?: number;
  classifiedFrom: string; // jobs/paths evidence, not conclusion alone
}

// Heartbeat context (extended)
interface HeartbeatContext {
  taskId: string;
  step: string; // now includes ci-poll, sandbox-deploy, sandbox-health,
  // verification, chronicle, digest
  stepElapsedMs: number;
  agentAlive: boolean; // this run's dispatched process tree only
  convergence?: {
    lastCommitAgeMs: number;
    diffDeltaLines: number;
    budgetRemainingMs: number;
    thrashWarning: boolean;
  };
}

// ETA band (status + digest)
interface EtaBand {
  optimisticMs: number;
  expectedMs: number;
  pessimisticMs: number;
  basis: string; // e.g. "12 merged S-complexity tasks in this repo"
}
```

## 5. Constraints & Dependencies

- Platform boundary: assertion _content_ (URLs, paths, artifact names) lives
  only in consumer `.sdlc/` scripts; the engine ships the schema, the
  invocation, and the classification logic. The companion
  `comita_admissions` PR is referenced by, but outside, the engine spec's
  envelope (the SPEC-PRD-0011-P4 pattern).
- Depends on SPEC-PRD-0011-P4 (path-aware sandbox deploy) — completed and
  live-validated inside this PRD's first phase.
- PRD-0021's recovery records and PRD-0020's wake path are consumers of the
  observability surface; interfaces coordinated but neither is a hard
  prerequisite.
- ADR-0004 stays authoritative for production observability; this PRD's
  scope ends at the sandbox delivery verdict and the run's own progress.
- HSR + InversifyJS, TypeScript strict; Chronicle artifacts per ADR-0007.

## 6. Risks & Open Questions

- Content assertions add per-phase latency (fetches against the live
  sandbox); bounded by running them once per deployed SHA, not per task.
- ETA bands from thin history will be wide and possibly wrong-footed early;
  the band must always carry its evidence basis, and "insufficient history"
  is an honest answer.
- Thrash detection thresholds risk killing legitimately slow exploratory
  work; the monitor warns and surfaces — it never terminates on its own
  (termination stays with PRD-0020's stale-agent policy and its explicit
  staleness rules).
- Deploy classification depends on workflow job introspection that varies
  by consumer CI shape; the contract must define what the repo exposes
  rather than the engine parsing arbitrary workflows.

## 7. Rollout & Phases

1. **Phase 1 — Path-aware deploy landed + deploy dedup:** complete and
   live-validate SPEC-PRD-0011-P4; content-SHA deploy records; merge-path
   reuse; remove the racing phase-boundary dispatch.
2. **Phase 2 — Content-level delivery verdicts:** contract schema + engine
   invocation and deploy classification; companion `comita_admissions`
   assertion scripts; sandbox verdict carries fetch evidence; negative
   assertions block delivery.
3. **Phase 3 — Honest run surface:** heartbeat step-context coverage,
   per-run `agentAlive`, convergence monitor with thrash warnings, `status`
   and digest with per-task step / remaining gates / recovery state / ETA
   bands from Chronicle history.

## 8. Future Considerations

- Distilling stable content assertions into the consumer's scripted E2E
  regression floor (PRD-0011 §8 progressive trust).
- Cross-surface auth-flow verification as a consumer E2E suite seeded from
  the recurring accounts/admit `redirect_uri` regression class.
- Per-track sandboxes (Comita PRD-0008) inherit the same delivery contract
  unchanged — the assertions are already repo-owned.
- Production delivery verdicts (promotion checklist automation) once the
  sandbox loop has track record.
