---
id: PRD-0011
title: Full-Loop SDLC Automation
status: Accepted
date: 2026-07-25
owner: Russ Watson
related_adrs: [ADR-0007, ADR-0008]
related_specs: [SPEC-PRD-0011-P1, SPEC-PRD-0011-P2, SPEC-PRD-0011-P3]
supersedes: null
---

# PRD-0011: Full-Loop SDLC Automation

> A dynamic multi-agent workflow that takes an epic from intake to a merged,
> sandbox-deployed milestone — decomposing it into product stories, engineering
> tasks, and a full implementation spec, then implementing, verifying, reviewing,
> and merging each phase autonomously — with human judgment front-loaded at spec
> approval and reserved for promotion beyond the sandbox.

## 1. Overview & Goals

### 1.1 Purpose

Today the Rosetta SDLC loop is sequential and turn-by-turn: each phase requires a
human prompt, and mechanical work (story decomposition, task breakdown, PR review
cycles, CI monitoring) runs at conversation speed. As Rosetta's scope grows, this
friction compounds: the gap between "epic approved" and "code merged" is dominated
by bookkeeping, not judgment.

The missing piece is a **deterministic orchestration layer** that compresses all
the mechanical work between human judgment calls into a single, auditable,
resumable workflow run. A gate is only justified where a machine cannot verify
the decision. Watson's judgment is front-loaded into one approval — the
implementation spec and its blast-radius envelope — and reserved at the end for
promotion beyond the sandbox. Every phase boundary in between is an **evidence
gate**: executable acceptance criteria, CI, and independent reviewer-agent
concurrence decide whether the run advances, and humans are pulled in only by
exception.

### 1.2 Goals

- Take a single epic or PRD as input and produce a merged, tested,
  sandbox-deployed milestone with no per-step human prompting.
- Decompose epics into product stories and engineering tasks automatically, using
  the existing PRD / implementation-spec format as the contract.
- Generate the implementation spec — including per-phase executable acceptance
  criteria and a blast-radius envelope — and surface it for Watson's single
  up-front approval before a line of code is written.
- Replace milestone checkpoints with machine gates: verification suite green, CI
  green, independent reviewer-agent concurrence, and envelope compliance
  auto-advance the run.
- Verify each phase against its acceptance criteria in the running sandbox,
  including agent-driven use of the actual interface (see §3, Verification
  tiers).
- Post a digest to the personal queue (PRD-0007) at each phase boundary —
  approval is the absence of veto, and a veto triggers revert.
- Capture the workflow run, per-phase verdicts, and outputs as Chronicle
  artifacts so the org knowledge base grows as a side-effect of shipping and
  gate policy can learn from track record over time.

### 1.3 Non-Goals

- Does not remove human judgment — it front-loads it. The spec + envelope
  approval and the promotion gate beyond the sandbox are load-bearing.
- Does not auto-promote beyond the sandbox environment; promotion toward
  production is always a human decision.
- Does not auto-approve PRs that fail CI or reviewer-agent concurrence; failing
  checks always escalate for human triage.
- Does not generate new PRDs autonomously; the epic/PRD is always a human-authored
  input.
- Does not handle cross-repo dependency ordering (initial scope: single-repo epics).
- Does not replace the existing SDLC loop for small tasks where a workflow would be
  over-engineered.

### 1.4 Acceptance Criteria

- [ ] Given a PRD ID, the workflow produces a structured implementation spec
      (phases, tasks, executable acceptance criteria, blast-radius envelope)
      and pauses exactly once for Watson's approval before implementation.
- [ ] After spec approval, implementation agents run in parallel per phase/task,
      each in worktree isolation, with no shared state conflicts.
- [ ] Each phase boundary auto-advances when all machine gates pass:
      acceptance-criteria verification suite green, CI green, independent
      reviewer-agent concurrence, and envelope compliance.
- [ ] Acceptance criteria are verified against the running sandbox, including
      agent-driven interface verification, with evidence (test output, agent
      transcript, screenshots) attached to each verdict.
- [ ] Each merged phase auto-deploys to the sandbox and posts a digest to the
      personal queue (PRD-0007); a veto triggers revert.
- [ ] Watson is interrupted only by exception: reviewer-agent disagreement, a
      third failing CI fix attempt, envelope breach, or budget exhaustion.
- [ ] The workflow is resumable: a killed or paused run re-uses cached agent results
      and only re-runs changed/new steps.
- [ ] Workflow outputs (implementation spec, per-phase verdicts, per-task
      results, merged SHAs) are committed to the Chronicle as structured
      artifacts.
- [ ] Token budget can be specified at invocation (e.g. `+500k`) to scale the
      depth of decomposition and verification.

## 2. Users & Motivation

**Primary user: Watson (and future Rosetta-native engineers).** The pain being
removed is the gap between a well-specified PRD and a merged milestone. That gap
is currently filled with mechanical, interruptive turn-by-turn prompting that
breaks focus and slows delivery.

**Secondary user: the Rosetta platform itself.** Demonstrating that Rosetta can
close the full engineering loop — from intent to shipped code — is a core part of
the product story. This capability is both a feature and proof-of-concept for what
Rosetta makes possible.

## 3. Approach

The workflow front-loads human judgment into a single approval, then runs
evidence-gated to a sandbox-deployed milestone:

```
[INPUT]  PRD / epic description
   │
   ▼
Phase 1 — Decompose & Specify
  ├── Parse PRD → extract features / stories
  ├── parallel: for each story → draft product story (user-story format)
  ├── pipeline(stories):
  │     stage 1: generate engineering tasks + acceptance criteria
  │     stage 2: score complexity + surface dependencies
  └── synthesize → implementation spec (phased) + blast-radius envelope

[HUMAN GATE — Watson approves spec + envelope. The last human word before code.]

Phase 2 — Implement (per spec phase)
  ├── stage 1: branch + implement (worktree isolation per task)
  ├── stage 2: verification suite — executable acceptance criteria, incl.
  │            agent-driven interface verification in the sandbox
  ├── stage 3: open PR → independent reviewer-agent pass
  ├── stage 4: CI monitoring
  ├── stage 5: auto-merge + auto-deploy to sandbox
  └── stage 6: digest → personal queue (PRD-0007); veto triggers revert

[MACHINE GATE at each phase boundary — advance when ALL hold:
  verification suite green · CI green · reviewer concurrence · envelope respected]

[HUMAN GATE — promotion beyond the sandbox (staging/prod). Outside workflow scope.]
```

Escalation replaces checkpoints: Watson is interrupted only on reviewer-agent
disagreement, a third failing CI fix attempt, envelope breach, or budget
exhaustion. Everything else advances on evidence.

### Verification tiers

Two complementary layers verify each phase; they are deliberately not the same
thing:

- **Scripted E2E tests** (e.g. Playwright) are deterministic regression code:
  pre-written paths through the real interface, cheap to run in CI, catching
  breakage of what already worked. The intelligence is applied at authoring
  time.
- **Agent-driven acceptance verification** applies judgment at runtime: an agent
  is handed the phase's acceptance criteria and a running sandbox, uses the
  interface as a user would (navigating, clicking, observing), and returns a
  per-criterion verdict with evidence attached. It validates new behavior
  directly against the spec with no pre-written test code.
- Stable agent verification runs are **distilled into scripted E2E tests**, so
  each phase grows the deterministic regression floor for the next — trust
  compounds (see §8, progressive trust).

### Script shape (illustrative)

```javascript
phase("Decompose & Specify");
const stories = await agent("Parse PRD and extract product stories", {
  schema: STORIES_SCHEMA,
});
const spec = await pipeline(
  stories,
  generateTasks,
  scoreDependencies,
  synthesizeSpec,
);
// [HUMAN GATE — surface spec + envelope, await approval]

phase("Implement");
for (const p of spec.phases) {
  await pipeline(
    p.tasks,
    implement,
    verifyAcceptance, // suite + agent-driven interface verification
    openPR,
    reviewerCycle,
    ciCycle,
    autoMerge,
  );
  await deployToSandbox(p);
  await postDigest(p); // PRD-0007 queue; veto → revert
  assertMachineGate(p); // suite · CI · concurrence · envelope
}

phase("Chronicle");
await agent(
  "Commit workflow outputs and per-phase verdicts as Chronicle artifacts",
);
```

## 4. Data Contracts

```typescript
// Input
interface WorkflowInput {
  prdId: string; // e.g. 'PRD-0011'
  repo: string; // e.g. 'rosetta_chronicle'
  budgetK?: number; // token budget in thousands (default: 200)
}

// Story (output of Decompose phase)
interface ProductStory {
  id: string; // e.g. 'S-01'
  title: string;
  asA: string;
  iWant: string;
  soThat: string;
  acceptanceCriteria: string[];
}

// Spec task (output of Decompose & Specify phase)
interface SpecTask {
  storyId: string;
  phase: number;
  title: string;
  engineeringNotes: string;
  complexity: "S" | "M" | "L";
  dependsOn: string[]; // task IDs
  acceptanceCriteria: string[];
}

// Blast-radius envelope (approved together with the spec)
interface Envelope {
  allowedPaths: string[]; // globs the run may modify
  forbiddenSurfaces: string[]; // e.g. 'migrations', 'auth', 'ci-config'
  maxDiffLines: number;
  budgetK: number;
}

// Per-phase verdict (output of each machine gate)
interface PhaseVerdict {
  phase: number;
  criteria: Array<{
    criterion: string;
    verdict: "pass" | "fail";
    evidence: string; // test run, agent transcript, screenshot ref
  }>;
  reviewerConcurrence: boolean;
  ciGreen: boolean;
  envelopeRespected: boolean;
  advanced: boolean; // false → escalated to Watson
}

// Per-task result (output of Implement phase)
interface TaskResult {
  taskId: string;
  branch: string;
  prUrl: string;
  mergedSha: string | null; // null if failed + surfaced for human review
  status: "merged" | "needs-review";
}
```

## 5. Constraints & Dependencies

- Requires Claude Code dynamic workflows (`Workflow` tool) — Claude Code CLI, not
  just the API.
- Requires a deployable sandbox environment per target repo; auto-deploy on
  merge is a prerequisite for agent-driven verification and the veto model.
- Agent-driven interface verification requires browser/automation access to the
  sandbox from the workflow runtime.
- Worktree isolation per implementation agent requires git worktree support (already
  confirmed in `rosetta_chronicle` and `rosetta_wayfinder`).
- All AI calls route through the Claude API (`ANTHROPIC_API_KEY`) per ADR-0003.
- HSR pattern applies to any TypeScript orchestration helpers that are extracted
  from workflows into Rosetta services.
- Depends on PRD-0007 (queue) for the phase-boundary digests and veto surface.
- Depends on PRD-0009 (coherence protocol) for the Chronicle artifact commit step.
- Token cost: a typical 6-story epic with parallel implementation runs ~200K–500K
  tokens. Human must specify a budget at invocation.

## 6. Risks & Open Questions

- **Veto window mechanics:** digests post to the personal queue (PRD-0007).
  Initial position: the veto blocks nothing — the run advances and a veto
  triggers revert (cheap in the sandbox). Does any class of change warrant a
  blocking window instead?
- **Dependency ordering in parallel implementation:** Stories with cross-task
  dependencies must be sequenced. The complexity-scoring stage needs to produce
  an ordering that the pipeline respects.
- **Partial failures:** If 3 of 6 tasks merge cleanly and 3 need review, the
  workflow should surface only the failing ones without blocking the passing ones.
- **Scope creep in decomposition:** Agents may over-decompose a small PRD.
  The spec review gate is the safety valve, but guidance on "right-sizing" stories
  should be in the workflow prompt.
- **Agent-driven verification flakiness and cost:** runtime judgment is
  non-deterministic. Verdicts must carry evidence, and repeated verification
  failures should escalate to human triage rather than loop.
- **Envelope authoring:** too tight and every phase escalates; too loose and the
  gate protects nothing. Start conservative and let Chronicle track record
  loosen it over time (see §8).

## 7. Rollout & Phases

1. **Phase 1 — Decompose + spec generation:** Workflow takes a PRD, produces a
   structured story list, implementation spec, and envelope; pauses at the
   single human gate. No implementation. Validates decomposition and envelope
   quality before any code is written.

2. **Phase 2 — Single-task loop with shadow gates:** Extend the workflow to
   implement one task end-to-end (branch → implement → verify → PR → reviewer →
   CI → auto-merge → sandbox deploy). Machine gates compute their verdicts, but
   Watson still confirms each boundary. Calibrates gate trustworthiness before
   removing checkpoints.

3. **Phase 3 — Full parallel implementation with live machine gates:** Fan out
   across all spec tasks in worktree isolation. Auto-advance on green, digest +
   veto via PRD-0007, sandbox deploy per merged phase, Chronicle artifact
   commit, PRD-0007 integration for next-phase surfacing.

## 8. Future Considerations

- **Cross-repo epics:** Phase 1 scopes to single-repo. Multi-repo coordination
  (e.g. a Chronicle + Wayfinder change in the same epic) is a natural follow-on.
- **Progressive trust via Chronicle:** per-repo / per-complexity track record
  tunes gate strictness — after N clean runs, "S" tasks skip shadow
  confirmation entirely; "L" tasks keep tighter envelopes and stricter review.
- **E2E distillation:** automatically promote stable agent verification runs
  into the scripted E2E suite, so the regression floor grows with every phase.
- **Automated PRD authoring:** If the coherence protocol (PRD-0009) matures enough,
  epics could be seeded from Chronicle observations rather than written by hand.
- **ADR generation:** At implementation time, when a significant technical decision
  is made, the workflow could draft the ADR automatically and surface it for
  Watson's approval alongside the implementation spec.
- **Budget-aware decomposition:** Scale story count and verification depth
  dynamically to the token budget rather than requiring a static directive.
