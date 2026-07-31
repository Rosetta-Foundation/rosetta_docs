---
id: PRD-0011
title: Full-Loop SDLC Automation
status: Proposed
date: 2026-07-25
owner: Russ Watson
related_adrs: []
related_specs: []
supersedes: null
---

# PRD-0011: Full-Loop SDLC Automation

> A dynamic multi-agent workflow that takes an epic from intake to merged milestone
> — decomposing it into product stories, engineering tasks, and a full SDD, then
> implementing, reviewing, and merging each phase in parallel — with Watson as the
> only required human at milestone boundaries.

## 1. Overview & Goals

### 1.1 Purpose

Today the Rosetta SDLC loop is sequential and turn-by-turn: each phase requires a
human prompt, and mechanical work (story decomposition, task breakdown, PR review
cycles, CI monitoring) runs at conversation speed. As Rosetta's scope grows, this
friction compounds: the gap between "epic approved" and "code merged" is dominated
by bookkeeping, not judgment.

The missing piece is a **deterministic orchestration layer** that compresses all
the mechanical work between human judgment calls into a single, auditable,
resumable workflow run. Watson's judgment is preserved at the two gates that
actually require it — SDD review before implementation starts, and milestone
review before the next phase is authorized. Everything else is automated.

### 1.2 Goals

- Take a single epic or PRD as input and produce a merged, tested milestone with
  no per-step human prompting.
- Decompose epics into product stories and engineering tasks automatically, using
  the existing PRD/SDD format as the contract.
- Generate the SDD and surface it for Watson's review before a line of code is
  written.
- Run implementation, testing, PR creation, Copilot review cycle, CI monitoring,
  and auto-merge in parallel across tasks/stories where there are no dependencies.
- Capture the workflow run and its outputs as Chronicle artifacts so the org
  knowledge base grows as a side-effect of shipping.
- Integrate with PRD-0007 (personal queue) so the next phase surfaces automatically
  after each milestone merges.

### 1.3 Non-Goals

- Does not replace Watson's architectural or strategic judgment — the two human
  gates are load-bearing.
- Does not auto-approve PRs that fail CI or Copilot review; failing checks always
  surface for human triage.
- Does not generate new PRDs autonomously; the epic/PRD is always a human-authored
  input.
- Does not handle cross-repo dependency ordering (initial scope: single-repo epics).
- Does not replace the existing SDLC loop for small tasks where a workflow would be
  over-engineered.

### 1.4 Acceptance Criteria

- [ ] Given a PRD ID, the workflow produces a structured SDD (phases, tasks,
      acceptance criteria) and pauses for Watson's approval before proceeding.
- [ ] After SDD approval, implementation agents run in parallel per phase/task,
      each in worktree isolation, with no shared state conflicts.
- [ ] Each agent runs the full SDLC finish sequence: tests → push → PR →
      Copilot review cycle → CI monitoring → auto-merge on clean.
- [ ] Failing CI or Copilot issues are surfaced to Watson rather than auto-resolved
      after the third fix attempt (matching the existing PR checks cycle).
- [ ] The workflow is resumable: a killed or paused run re-uses cached agent results
      and only re-runs changed/new steps.
- [ ] Workflow outputs (SDD, per-task results, merged SHAs) are committed to the
      Chronicle as structured artifacts.
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

The workflow is structured as three phases separated by two human gates:

```
[INPUT]  PRD / epic description
   │
   ▼
Phase 1 — Decompose
  ├── Parse PRD → extract features / stories
  ├── parallel: for each story → draft product story (user-story format)
  └── synthesize → structured story list (machine-legible JSON)

[HUMAN GATE 1 — Watson reviews story list]

Phase 2 — SDD Generation
  ├── pipeline(stories):
  │     stage 1: generate engineering tasks + acceptance criteria
  │     stage 2: score complexity + surface dependencies
  └── synthesize → full SDD draft (phased, Chronicle SDD format)

[HUMAN GATE 2 — Watson approves SDD before implementation]

Phase 3 — Implement
  └── pipeline(sdd.phases):
        stage 1: branch + implement (worktree isolation per task)
        stage 2: run tests
        stage 3: open PR
        stage 4: Copilot review cycle
        stage 5: CI monitoring
        stage 6: auto-merge if clean / surface failures if not
   → Chronicle artifact commit per merged phase
```

The two human gates are the only blocking checkpoints. All inter-gate work runs
as a single workflow invocation.

### Script shape (illustrative)

```javascript
phase("Decompose");
const stories = await agent("Parse PRD and extract product stories", {
  schema: STORIES_SCHEMA,
});
// [HUMAN GATE — surface stories, await approval before continuing]

phase("SDD Generation");
const sdd = await pipeline(
  stories,
  generateTasks,
  scoreDependencies,
  synthesizeSDD,
);
// [HUMAN GATE — surface SDD, await approval]

phase("Implement");
await pipeline(
  sdd.phases,
  implement,
  test,
  openPR,
  copilotCycle,
  ciCycle,
  autoMerge,
);

phase("Chronicle");
await agent("Commit workflow outputs as Chronicle artifacts");
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

// SDD task (output of SDD Generation phase)
interface SddTask {
  storyId: string;
  phase: number;
  title: string;
  engineeringNotes: string;
  complexity: "S" | "M" | "L";
  dependsOn: string[]; // task IDs
  acceptanceCriteria: string[];
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
- Worktree isolation per implementation agent requires git worktree support (already
  confirmed in `rosetta_chronicle` and `rosetta_wayfinder`).
- All AI calls route through the Claude API (`ANTHROPIC_API_KEY`) per ADR-0003.
- HSR pattern applies to any TypeScript orchestration helpers that are extracted
  from workflows into Rosetta services.
- Depends on PRD-0007 (queue) for surfacing the next phase after a milestone merges.
- Depends on PRD-0009 (coherence protocol) for the Chronicle artifact commit step.
- Token cost: a typical 6-story epic with parallel implementation runs ~200K–500K
  tokens. Human must specify a budget at invocation.

## 6. Risks & Open Questions

- **Human gate mechanics:** How exactly are the two gates implemented? Options:
  (a) workflow pauses and sends a summary message, Watson replies "approved";
  (b) workflow writes an artifact and Watson triggers the next phase explicitly.
  To be decided at SDD time.
- **Dependency ordering in parallel implementation:** Stories with cross-task
  dependencies must be sequenced. The complexity-scoring stage needs to produce
  an ordering that the pipeline respects.
- **Partial failures:** If 3 of 6 tasks merge cleanly and 3 need review, the
  workflow should surface only the failing ones without blocking the passing ones.
- **Scope creep in decomposition:** Agents may over-decompose a small PRD.
  The SDD review gate is the safety valve, but guidance on "right-sizing" stories
  should be in the workflow prompt.

## 7. Rollout & Phases

1. **Phase 1 — Decompose + SDD generation:** Workflow takes a PRD, produces a
   structured story list and SDD, pauses at both human gates. No implementation.
   Validates the decomposition quality before any code is written.

2. **Phase 2 — Single-task implementation loop:** Extend the workflow to implement
   one task end-to-end (branch → implement → tests → PR → Copilot → CI →
   auto-merge). Human gate before and after. Validates the full loop on a
   single task before going parallel.

3. **Phase 3 — Full parallel implementation:** Fan out across all SDD tasks in
   worktree isolation. Surface failures. Chronicle artifact commit. PRD-0007
   integration for next-phase surfacing.

## 8. Future Considerations

- **Cross-repo epics:** Phase 1 scopes to single-repo. Multi-repo coordination
  (e.g. a Chronicle + Wayfinder change in the same epic) is a natural follow-on.
- **Automated PRD authoring:** If the coherence protocol (PRD-0009) matures enough,
  epics could be seeded from Chronicle observations rather than written by hand.
- **ADR generation:** At implementation time, when a significant technical decision
  is made, the workflow could draft the ADR automatically and surface it for
  Watson's approval alongside the SDD.
- **Budget-aware decomposition:** Scale story count and verification depth
  dynamically to the token budget rather than requiring a static directive.
