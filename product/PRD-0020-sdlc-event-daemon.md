---
id: PRD-0020
title: SDLC Event Daemon
status: Accepted # Draft | Proposed | Accepted | Shipped | Superseded | Deprecated
date: 2026-08-04
owner: Russ Watson
related_adrs: [ADR-0007, ADR-0008]
related_specs:
  [
    'https://github.com/Comita-Health/rosetta_dev-scripts/blob/main/specs/PRD-0020/phase-1-spec.md',
    'https://github.com/Comita-Health/rosetta_dev-scripts/blob/main/specs/PRD-0020/phase-2-spec.md',
    'https://github.com/Comita-Health/rosetta_dev-scripts/blob/main/specs/PRD-0020/phase-3-spec.md',
  ]
supersedes: null
---

# PRD-0020: SDLC Event Daemon

> A session-independent daemon that owns the entire "something happened →
> the machinery acts" chain for SDLC automation: GitHub event watching, wake
> delivery, supervisor continuity, and headless agent dispatch — replacing
> session-mortal watchers and the bash continuity daemon.

## 1. Overview & Goals

### 1.1 Purpose

The watcher → wake → agent-turn chain is the weakest link in the automated
SDLC: it broke in five distinct layers in one week of live runs. Skill
watchers are bash poll loops that die with the chat session that launched
them; two of the three bypass the durable wake inbox entirely; the only
deterministic agent re-entry (the editor stop hook) fires only when a turn
ends, so wakes rot while chat is idle; and the launchd continuity daemon is a
414-line bash script that re-implements engine logic, hardcodes a sibling
workspace's path, and resurrects only run supervisors — never watchers. This
PRD replaces all of it with one session-independent event daemon owned by the
engine.

### 1.2 Goals

- No SDLC signal (PR approve, request-changes, review comment, CI terminal
  state, issue close, deploy completion, run heartbeat gap, supervisor death)
  ever depends on a chat session being open to be observed or acted on.
- One durable delivery path: every event lands in the wake inbox exactly
  once, survives process and machine restarts, and is observable ("what is
  being watched right now?" has a machine answer).
- Wake consumption without a human: wakes can dispatch a headless operator
  agent turn so approved work continues while the operator is away; chat
  notification becomes a best-effort mirror, never load-bearing.
- Continuity behaviors (dead-supervisor relaunch, stale-agent kill,
  blocker-close resume) run as daemon modules sharing the engine's own state
  readers — no logic re-implementation in shell.
- Fully workspace-agnostic: one daemon instance per workspace, configured by
  a declared workspace root; zero hardcoded orgs, repos, paths, or
  domain-specific rules.

### 1.3 Non-Goals

- Not a chat bot or conversational surface — the daemon never holds a
  conversation; it delivers events and dispatches work.
- Does not replace the human gates: spec approval and sandbox smoke/veto
  remain human acts; the daemon only makes their consequences automatic.
- Does not learn deploy/test mechanics — repo `.sdlc/` contracts still own
  those (the daemon watches outcomes, it does not perform deploys).
- Does not implement the one-click Draft → Approved flip itself (that is a
  separate reusable workflow; the daemon consumes its launch signal).
- No cross-machine or hosted operation — single operator workstation,
  launchd-managed, matching the local-first posture.

### 1.4 Acceptance Criteria

- [ ] A watch registered for a PR survives the end of the chat session that
      registered it: with no editor open, an Approve on that PR produces a
      consumed wake and the configured follow-up action within one poll
      interval.
- [ ] `sdlc-workflow daemon status` lists every active watch (kind, target,
      age, last poll) and every pending/consumed wake; an unwatched PR or run
      is visible as such.
- [ ] Killing a detached run supervisor mid-wave results in automatic
      relaunch within one daemon tick, with the relaunch recorded in the run's
      monitor log and a wake emitted.
- [ ] A wake with a registered headless action dispatches a non-interactive
      agent run whose transcript is persisted; no chat turn is required.
- [ ] Two workspaces on the same machine run independent daemon instances
      with disjoint inboxes and watch registries; events from one never wake
      the other.
- [ ] The bash continuity daemon and the three session-mortal watcher scripts
      are retired; their launchd entry is replaced by the per-workspace daemon
      plist.
- [ ] Every daemon failure mode is loud: a dead daemon is detected by its own
      launchd keepalive; a poll error emits an operator-visible wake after
      bounded retries; nothing exits 0 on failure.

## 2. Users & Motivation

**Primary user: the operator (Watson).** Removes the dominant babysitting
mode observed across six sessions (Jul 31 – Aug 4): discovering stalls by
asking "what's going on?", re-arming dead watchers, manually merging because
"chat 'Approved' + me merging manually is faster," and returning overnight to
unconsumed wakes. The operator's two touchpoints (Approve, smoke) stay; every
other continuation becomes the daemon's job.

**Secondary user: the engine and its agents.** Escalation issues, heartbeat
gaps, and blocker closures get a guaranteed consumer instead of depending on
a chat session that may have ended hours ago.

## 3. Approach

One TypeScript daemon inside `sdlc-workflow` (Handler / Service / Repository

- InversifyJS, like the rest of the engine), run per workspace under launchd
  with `KeepAlive`. Modules:

* **Watch registry** — durable registrations (`~/.rosetta/<workspace>/daemon/
watches/`) created by the engine (run start, PR open, escalation filed) or
  by operator skills; each watch declares kind, target, poll cadence, and
  follow-up action.
* **GitHub poller** — one poller for all watches (PR reviews, check runs and
  status contexts, issue state, workflow runs), using the workspace's Addi
  activate script for identity with token refresh; per-target dedup keys.
* **Wake inbox (unified)** — the existing durable inbox becomes the daemon's
  single transport, scoped per workspace; the editor stop hook and macOS
  notifications remain as mirrors.
* **Headless dispatcher** — a wake whose watch declares an action dispatches
  a non-interactive agent (`agent -p` / configured runner) with a scoped
  prompt and persisted transcript, so continuation does not wait for chat.
* **Continuity module** — the bash daemon's behaviors reimplemented against
  the engine's own state readers: relaunch dead supervisors from
  `launch.json`, kill stale implementation agents (per-run identity, not
  machine-global `pgrep`), wake on blocker-issue close, flag abandoned runs.
* **Status surface** — `sdlc-workflow daemon status` (and `daemon install` /
  `daemon uninstall` for the plist) answering coverage questions from CLI
  and from GitHub (a scheduled digest comment is a later phase).

Placement: engine and daemon are upstream (Rosetta-Foundation) and
domain-agnostic; each consumer workspace provides only config —
workspace root, activate script path, watched repos.

## 4. Data Contracts

```ts
// Watch registration (durable, one file per watch)
interface WatchRegistration {
  id: string; // stable key: kind + target
  kind:
    | "pr-review" // Approve / Request changes / new review comments
    | "pr-checks" // CI terminal states, status contexts
    | "issue-state" // needs-human blocker close, intake labels
    | "workflow-run" // deploy completion
    | "run-supervisor" // detached run liveness + heartbeat gap
    | "queue-item"; // veto tags on digests
  target: { repo?: string; number?: number; runId?: string };
  pollSeconds: number;
  action?: HeadlessAction; // absent = deliver wake only
  createdBy: string; // engine step, skill, or operator
  expiresAt?: string; // ISO; terminal states auto-expire
}

interface HeadlessAction {
  kind: "agent-dispatch" | "engine-command";
  prompt?: string; // for agent-dispatch, scoped instruction
  argv?: string[]; // for engine-command, e.g. run resume
  transcriptDir: string;
}

// Wake event (inbox file, one per event, dedup by id)
interface WakeEvent {
  id: string; // kind + target + signal, idempotent
  workspace: string; // absolute workspace root
  kind: WatchRegistration["kind"];
  signal: string; // e.g. "approved", "checks-failed", "supervisor-dead"
  payload: Record<string, unknown>; // links, SHAs, verdicts
  emittedAt: string;
  consumedBy?: string; // stop-hook | headless-dispatch | cli
}

// Daemon config (per workspace, consumer-owned)
interface DaemonConfig {
  workspaceRoot: string;
  activateScript: string; // Addi identity for gh calls
  runsDir: string; // engine run state location
  defaultPollSeconds: number;
  headlessRunner: string; // e.g. "cursor-agent"
}
```

## 5. Constraints & Dependencies

- HSR + InversifyJS architecture, TypeScript strict — same bar as the rest of
  the engine; the continuity module must reuse `run-completion` /
  run-state readers rather than duplicating their logic.
- Workspace-agnostic: no consumer path, org, label, or
  hostname in daemon code; consumer opinions arrive only via `DaemonConfig`.
- Identity: all GitHub calls under the workspace's Addi App token (activate
  script), with the fork-targeting diagnostic honored before any PR/issue
  write.
- Depends on the engine's existing run artifacts (`state.json`,
  `launch.json`, `heartbeat.jsonl`) and the wake-inbox layout it absorbs.
- The one-click spec-approval workflow (separate bug-spec) emits the launch
  signal this daemon consumes; sequencing: daemon Phase 1 must land first.
- launchd (macOS) is the process supervisor; the daemon does not implement
  its own keepalive.
- Healthcare guardrails: the daemon never captures PHI; wake payloads carry
  links and SHAs, not application data.

## 6. Risks & Open Questions

- Headless dispatch cost and safety: a wake storm could dispatch many agent
  runs; per-kind concurrency caps and dedup keys are required from day one.
- GitHub API rate limits with one poller across many watches; mitigation is
  conditional requests (ETags) and per-watch cadence tuning.
- The stop-hook mirror can double-deliver with headless dispatch; the
  `consumedBy` claim (atomic rename) must make consumption exactly-once.
- Migration window: bash daemon and new daemon must not both relaunch the
  same supervisor; cutover disables the old plist before the new one arms.
- Is `launchd` the right floor for Linux contributors later? Out of scope
  now; the process-supervisor boundary is isolated behind `daemon install`.

## 7. Rollout & Phases

1. **Phase 1 — Daemon core + watch registry + unified inbox:** `sdlc-workflow
daemon` command with launchd install; watch registry; GitHub poller
   covering `pr-review` and `pr-checks`; per-workspace inbox scoping;
   `daemon status`. The pr-approve watcher script is absorbed (skill becomes
   a thin register/status client).
2. **Phase 2 — Continuity fold-in:** supervisor relaunch, per-run stale-agent
   kill, blocker-close wake, abandoned-run flagging as daemon modules using
   engine state readers; retire `sdlc-continuity-daemon.sh` and its plist;
   loud-failure semantics (keepalive, bounded-retry poll errors).
3. **Phase 3 — Full watch coverage + headless dispatch:** `issue-state`,
   `workflow-run`, `run-supervisor`, `queue-item` watches (deploy-verify and
   issue-resolve scripts retired; skills become clients); headless action
   dispatch with transcripts; scheduled coverage digest.

## 8. Future Considerations

- GitHub webhooks (via a local tunnel or repo-dispatch relay) replacing
  polling where latency matters.
- A pipeline-of-plans scheduler on top of the watch registry: plan B queues
  behind approved run A and launches on A's completion wake.
- Cross-workspace federation (one status view over multiple consumer daemons).
- Linux/systemd support behind the `daemon install` boundary.
