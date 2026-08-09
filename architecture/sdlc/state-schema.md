# `state.json` schema

The complete durable state of one run. Everything the engine knows that survives a
process boundary is here, which makes this file the first thing to read when a run
did something you did not expect.

Source: `RunState` in [`src/types.ts`](https://github.com/Rosetta-Foundation/rosetta_dev-scripts/blob/main/sdlc-workflow/src/types.ts),
written exclusively by [`repositories/run-state.repository.ts`](https://github.com/Rosetta-Foundation/rosetta_dev-scripts/blob/main/sdlc-workflow/src/repositories/run-state.repository.ts).

Location: `<runsDir>/<runId>/state.json`, default
`~/.rosetta/sdlc-runs/<runId>/state.json`.

## Write discipline

Two properties hold for every write, and both were added in response to real
corruption:

- **Atomic.** Temp file → fsync → rename, so a kill mid-write leaves the previous
  valid state rather than a truncated JSON file.
- **Single-writer.** A sibling `run.lock` means a second process against the same
  run refuses to start. This is what makes continuity-daemon relaunch safe.

Older states are forward-migrated on load (missing fields default rather than
throwing), so a run started by an earlier engine version still reports status.

## Top-level fields

| Field                 | Type                                | Written by                      | Purpose                                                      |
| --------------------- | ----------------------------------- | ------------------------------- | ------------------------------------------------------------ |
| `runId`               | `string`                            | Launch                          | Identity; the run directory name                             |
| `specId`              | `string`                            | Launch                          | The spec being implemented                                   |
| `specPath`            | `string`                            | Launch                          | Absolute path to the spec document                           |
| `baseSha`             | `string`                            | Launch                          | Frozen run baseline                                          |
| `startedAt`           | `string?`                           | Launch, before intake           | So a crash mid-intake still has a describable run            |
| `specDigest`          | `string?`                           | Intake                          | Digest of the spec as executed                               |
| `launchArgv`          | `string[]?`                         | Launch                          | Forensics: exactly how this run was invoked                  |
| `taskResults`         | `Record<taskId, TaskRunResult>`     | Executor, merge, `record-merge` | Per-task status, branch, PR, merge SHA                       |
| `verdicts`            | `GateVerdict[]`                     | Every gate step                 | Append-only verdict history                                  |
| `exceptions`          | `ExceptionEntry[]`                  | Aggregator via handler          | Escalation triggers raised                                   |
| `criterionVerdicts`   | `CriterionVerdict[]`                | Verification gate               | Per-criterion outcomes — the input to closeout               |
| `steps`               | `Record<stepKey, StepResult>`       | Every step                      | The step cache; `name:taskId:inputsDigest`                   |
| `sandbox`             | `SandboxRecord?`                    | Sandbox gate                    | Last deploy: SHA, content SHA, health status                 |
| `mergedSha`           | `string?`                           | Merge / `record-merge`          | Run-level merge (phase-scoped merges live per task)          |
| `tokenSpendK`         | `number`                            | Agent dispatch                  | Cumulative model spend in thousands, metered where available |
| `ciFixAttempts`       | `Record<taskId, number>`            | CI gate                         | Failing-CI fix attempts; cap 3                               |
| `gateFixAttempts`     | `Record<taskId, number>`            | Gate remediation                | Reviewer/envelope re-dispatch rounds                         |
| `remediations`        | `Record<taskId, RemediationRecord>` | Gate remediation                | Latest remediation per task, for re-selection                |
| `mergeBlockedRetries` | `number`                            | Supervisor                      | Merge-blocked retry count for this run                       |
| `updatedAt`           | `string`                            | Every write                     | ISO timestamp                                                |

Three of these are budgets, and their being _persisted_ is the point:
`ciFixAttempts`, `gateFixAttempts`, and `mergeBlockedRetries` all exist so that a
resume cannot refill a budget the run already spent. An in-memory counter would make
every relaunch a fresh three attempts, which is an infinite loop with extra steps.

## `taskResults`

```ts
interface TaskRunResult {
  taskId: string;
  status: "completed" | "failed" | "blocked";
  branch?: string;
  worktreePath?: string;
  detail?: string;
  inputsDigest?: string; // the implementation digest this attempt ran against
  mergedSha?: string;
  prUrl?: string;
  recordedAt: string;
}
```

`status` and `mergedSha` answer different questions and are routinely confused.
`status: 'completed'` means the implementation agent finished; it says nothing about
gates. Dependency eligibility reads `mergedSha` only, which is why a completed task
with a red phase gate correctly blocks its dependents by staying unmerged.

## `verdicts`

```ts
interface GateVerdict {
  gate: string; // 'envelope' | 'reviewer' | 'sandbox' | 'verification' | 'ci' | 'phase' | 'intake'
  taskId?: string; // absent for run-level verdicts
  outcome: "pass" | "breach" | "blocked" | "human-required";
  wouldEscalate: boolean;
  reasons: string[];
  transcript?: string;
  inputsDigest?: string;
  evidenceIds?: string[];
  checklistFindings?: ChecklistFinding[];
  recordedAt: string;
}
```

Append-only: a re-judged gate adds a verdict rather than replacing one, so the
history of what a gate said across remediation rounds stays readable. When you need
"the current answer," take the most recent verdict for that `(taskId, gate)`.

See [gate model](gate-model.md) for what each gate puts in `reasons` and how
`outcome` differs from `wouldEscalate`.

## `criterionVerdicts`

```ts
interface CriterionVerdict {
  taskId: string;
  criterion: string; // the criterion text, tier prefix included
  tier: "test" | "agent" | "manual" | "docs";
  outcome: "pass" | "fail" | "human-required";
  evidenceId?: string;
  recordedAt: string;
}
```

This array is what makes automated closeout possible. Because each criterion has an
individual recorded outcome, the closeout PR can tick exactly the boxes with a
passing verdict and explain every one it left alone — rather than the all-or-nothing
signal a gate-level verdict would give.

## `steps` — the step cache

```ts
type StepKey = `${name}:${taskId}:${inputsDigest}`;

interface StepResult {
  name: string; // 'implementation' | 'pr' | 'envelope' | … | 'digest-post'
  taskId: string;
  inputsDigest: string;
  verdict?: GateVerdict; // gate steps cache their verdict verbatim
  detail?: string; // small payloads, e.g. a sandbox health report
  recovery?: RecoveryHistory;
  completedAt: string;
}
```

The key is the whole mechanism. Each step derives a digest over its inputs — which
chains off the implementation digest and head SHA — so a resume reuses a step
whose inputs are unchanged and re-runs one whose inputs moved. A spec content edit
invalidates exactly the affected task's steps, no more.

`recovery` is absent on a first-attempt success, which means its _presence_ is
itself the signal that a step was flaky:

```ts
interface RecoveryHistory {
  path: string; // e.g. 'pr:T-01' — one budget per path
  attempts: Array<{
    attempt: number;
    action: "attempt" | "backoff" | "escalate";
    outcome: "succeeded" | "failed" | "exhausted" | "waited";
    detail?: string; // failure message, or backoff duration
    at: string;
  }>;
  escalated: boolean; // true only once the attempt cap was reached
}
```

That record is the durable answer to "why did this step take four minutes and six
tries," which previously existed only in whichever terminal the operator still had
open.

## `sandbox`

```ts
interface SandboxRecord {
  sha: string;
  status: "healthy" | "failed";
  recordedAt: string;
  contentSha?: string; // tree-content SHA of `sha`
}
```

`contentSha` is recorded alongside the commit so dedup can ask "is this _content_
live?" A merge commit and the PR head it merged have different commit SHAs and the
same tree, so commit-keyed dedup deployed identical builds twice.

## `remediations`

```ts
interface RemediationRecord {
  attempt: number;
  sha: string; // head SHA the remediation produced
  gates: string[]; // gate names the round was asked to address
  recordedAt: string;
}
```

Task re-selection reads this to reopen a task whose phase gate breached _before_ a
fix landed. Without it the breach would stay terminal and the fix would never be
judged — the engine would have written a correction and then refused to look at it.

## Reading state by hand

`sdlc-workflow status --run-id <id>` is the intended entry point and renders the
useful summary. When you need the raw file, three queries answer most questions:

```bash
# What merged?
jq '.taskResults | to_entries | map({task: .key, status: .value.status, merged: .value.mergedSha})' state.json

# Why did it stop?
jq '.exceptions' state.json

# What did the gates most recently say for one task?
jq '[.verdicts[] | select(.taskId == "T-02")] | group_by(.gate) | map(last)' state.json
```

For the terminal exit reason, read `supervise.exit` in the same directory rather
than inferring it from state — see
[run lifecycle](run-lifecycle.md#supervisor-exit-reasons).
