---
id: PRD-0001
title: Claude Code Source
status: Shipped
date: 2026-07-22
owner: Russ Watson
related_adrs: [ADR-0002]
related_specs: []
supersedes: null
---

# PRD-0001: Claude Code Source

> **Frozen / deprecated as capture (2026-08-25).** v0.1 `getActivity` →
> Daily Chronicle is historical record. Do not extend. See
> [chronicle-build-charter.md](../process/chronicle-build-charter.md).

> Turn Claude Code sessions into Chronicle activity — both by scraping past
> sessions and by appending live as a session wraps up.

## 1. Overview & Goals

### 1.1 Purpose

Git records _what changed_ and notes record _what an engineer chose to write
down_. Neither captures _how the work was reasoned through_ — the investigation,
alternatives, and decisions that happen inside Claude Code sessions and are
otherwise lost. This capability recovers that reasoning into the Daily Chronicle.

### 1.2 Goals

- Extract meaningful, evidence-backed activity from Claude Code session transcripts.
- Support **batch backfill** — scrape past sessions over a date range into daily logs.
- Support **live append** — append a session's summary to today's log as it ends.
- Keep output compact and summary-level, not a transcript dump.

### 1.3 Non-Goals

- Not storing or copying raw conversation content into the Chronicle.
- Not reading transcripts outside the target project's scope.
- Not building the daily CLI itself (separate capability; this source plugs into it).
- Not organizational publication — output stays in the private Chronicle (ADR-0002).

### 1.4 Acceptance Criteria

- [x] `ClaudeCodeRepository.getActivity(window, projectPath)` returns one `Activity` per in-window session, with `Evidence` referencing the session id and any PR links.
- [x] Session summary is taken from the transcript `ai-title`; sessions with neither a title nor substantive prompts are dropped.
- [x] Only the target project's transcript directory is read — never a global sweep of `~/.claude/projects/*`.
- [x] Re-running over the same window never duplicates a session's entry (stable id from session id).
- [x] The generated Chronicle gains a "Claude Sessions" section when session activity exists.
- [x] A Stop-hook path can append the just-ended session to today's file idempotently.

## 2. Users & Motivation

Serves the engineer first (their private second brain), and later managers and
agents consuming organizational knowledge. Removes the "I misremember and lose
the reasoning" pain: the _why_ behind the day's work is captured automatically.

## 3. Approach

`ClaudeCodeRepository` implements the existing source pattern
(`getActivity(window, projectPath?)` → `Activity[]`), reading per-session JSONL
transcripts stored under a per-project directory. One `Activity` per session:

- `summary` ← session `ai-title` (fallback: truncated first user prompt).
- `timestamp` ← first in-window content record.
- `evidence` ← `sessionId` + any `pr-link` records (repo, number, url).

Two run modes share this extraction:

|         | Batch backfill         | Live append            |
| ------- | ---------------------- | ---------------------- |
| Trigger | CLI / on demand        | Claude Code Stop hook  |
| Window  | any past range         | today                  |
| Scope   | all in-window sessions | the just-ended session |

## 4. Data Contracts

```ts
export interface IClaudeCodeRepository {
  getActivity(
    window: ChronicleWindow,
    projectPath?: string,
  ): Promise<Activity[]>;
}
```

Reuses existing `Activity` / `Evidence` / `ChronicleWindow` types (`src/types.ts`).

## 5. Constraints & Dependencies

- **Privacy (ADR-0002):** project-scoped reads only; summaries not transcripts; private-repo destination; opt-in.
- **Architecture:** follows Handler / Service / Repository + InversifyJS (resource access only in the repository).
- **Depends on:** the notes content-hash dedup primitive (already shipped) for idempotent append.
- **External:** Claude Code transcript format (`ai-title`, `pr-link`, per-record `timestamp`/`cwd`/`gitBranch`).

## 6. Risks & Open Questions

- Sessions without an `ai-title` (~30% observed) — truncated first prompt, or drop?
- Granularity: one Activity per session (proposed) vs. per notable sub-task.
- Cross-project sessions: scope by transcript directory (proposed) vs. per-record `cwd`.
- Live-append cadence: every session end vs. only on explicit request.
- Even project-scoped titles may name systems/people — acceptable for a private log; must be re-sanitized on any organizational publication.

## 7. Rollout & Phases

1. ✅ **Phase 1 — `ClaudeCodeRepository` (batch):** parse project transcripts → session `Activity[]`; wire into `ChronicleService`; add a "Claude Sessions" section. Answers "scrape past sessions."
2. ✅ **Phase 2 — Backfill CLI:** generate + persist over a past date range.
3. ✅ **Phase 3 — Stop-hook live append:** reuse Phase 1 extraction for the single ended session; append to today's file.

## 8. Future Considerations

- Richer extraction (decisions/alternatives) beyond the title, if summaries prove too thin.
- Feeding Claude-session activity into Wayfinder's "why was this done?" answers.
- The full design detail lives in the Chronicle repo's `docs/design/claude-code-source.md`.
