---
id: PRD-0005
title: Regeneration Clobber Guard
status: Accepted
date: 2026-07-23
owner: Russ Watson
related_adrs: [ADR-0002]
related_specs: []
supersedes: null
---

# PRD-0005: Regeneration Clobber Guard

> Refuse, by default, to overwrite a day's Chronicle when regeneration would
> silently drop activity that a prior run had captured — closing the clobber
> class for derived git and Claude-session activity.

## 1. Overview & Goals

### 1.1 Purpose

[PRD-0003](PRD-0003-notes-as-authoritative-input.md) protected **notes** from
being clobbered by making them authoritative input. But git commits and Claude
sessions are still *derived per run*: each regeneration rebuilds them from
whatever sources the run can currently see. If a run sees **fewer** sources than
a prior run — a narrower `--project` scope, a repo whose local `main` is behind
its remote, an unreachable transcript directory — it rebuilds the day with less
activity and overwrites the richer prior Chronicle. No error, no warning.

This is not hypothetical: it happened during PRD follow-up work. A backfill
re-run scoped to `~/projects/rosetta` dropped an entire sibling repo (a project
that lives under `~/projects`, not under `rosetta`) plus two Claude sessions,
silently overwriting the correct Chronicle. The data was only recovered because
a later live capture happened to run with the correct broad scope.

The structured sidecar (PRD-0002) already records exactly what a prior run
captured. This capability uses it as a safety check: if a fresh run's activity
is a **strict subset** of what the sidecar already holds, that's a regression —
refuse to persist unless the user explicitly forces it.

### 1.2 Goals

- Detect when a regeneration would drop activity present in the existing sidecar.
- Refuse to persist a strictly-smaller Chronicle by default; report exactly what would be dropped.
- Provide an explicit `--force` escape hatch for legitimately-shrinking regenerations (e.g. a repo was intentionally removed).
- Extend the "never silently clobber" guarantee from notes (PRD-0003) to git and session activity.

### 1.3 Non-Goals

- Not merging old and new activity — the guard blocks the destructive write; it does not union prior and fresh activity (that would resurrect genuinely-deleted work). The fix for a scope mistake is to re-run with the correct scope.
- Not fetching from remotes or scanning all branches — the guard reports the subset condition; correcting *why* the scope was narrow (stale local main, wrong `--project`) stays the operator's responsibility (see PRD-0002's discovery model).
- Not protecting against content *changes* to the same activity id — only against activity disappearing.

### 1.4 Acceptance Criteria

- [x] Before persisting, the handler compares the freshly-generated activity id set against the existing sidecar's activity id set for that date.
- [x] When the fresh set is a strict subset of the prior set (activity would be lost), the run refuses to persist and exits non-zero, listing each dropped activity (source, repo/session, summary).
- [x] A `--force` flag (CLI) / `force` input (handler) bypasses the guard and persists anyway.
- [x] When the fresh set is equal to or a superset of the prior set, the run persists normally (no false positives on legitimate additions).
- [x] When no prior sidecar exists, the guard is a no-op (first run for a day is never blocked).
- [x] The guard reads the prior set from the structured sidecar (source of truth), not the rendered Markdown.

## 2. Users & Motivation

Serves the engineer running backfills and the automated live-capture hook alike.
Removes a silent-data-loss hazard: a mistyped scope, a stale checkout, or an
offline transcript store can no longer quietly shrink a day's durable record.
The failure becomes loud and recoverable (refuse + report) instead of silent and
destructive (overwrite).

## 3. Approach

The comparison is pure set logic over activity ids, which the sidecar already
persists (PRD-0002). It belongs in the **handler**, which already orchestrates
read-prior → generate → persist:

1. Handler reads the existing sidecar for the date (already does this for `priorTags`).
2. After the service generates the fresh `DailyChronicle`, compare id sets:
   - `dropped = priorIds \ freshIds`
   - If `dropped` is non-empty **and** `freshIds ⊂ priorIds` (strict subset — nothing new to justify the shrink), the guard trips.
3. On trip: unless `force`, skip both sidecar and Markdown persistence, return a structured "clobber prevented" result listing the dropped activities; the CLI prints them and exits non-zero.
4. On `force`, or when fresh adds anything new, or when there's no prior sidecar: persist as normal.

Set logic lives in a pure util (`src/utils/`); the handler composes it. No
service or repository change is required beyond what PRD-0002 already provides.

## 4. Data Contracts

```ts
// Pure comparison over two activity-id sets.
export interface ClobberCheck {
  /** Activity present before but absent now. */
  dropped: Activity[];
  /** True when persisting would lose activity with nothing new to offset it. */
  wouldClobber: boolean;
}

// Handler input gains an explicit override.
export interface DailyChronicleInput {
  // ...existing fields...
  /** Bypass the clobber guard and persist even if activity would be dropped. */
  force?: boolean;
}

// Handler result signals a prevented clobber.
export interface DailyChronicleResult {
  chronicle: DailyChronicle;
  persisted?: PersistedChronicle;
  /** Set when the guard blocked persistence; lists what would have been dropped. */
  clobberPrevented?: ClobberCheck;
}
```

Reuses the existing `Activity` type and the structured sidecar from PRD-0002.

## 5. Constraints & Dependencies

- **Architecture:** Handler / Service / Repository + InversifyJS. Comparison is a pure util; the handler composes read → generate → guard → persist.
- **Depends on:** PRD-0002's structured sidecar (the record of prior activity) and PRD-0003's handler flow (already reads the sidecar for tags).
- **Backward compatibility:** no prior sidecar ⇒ no-op; equal/superset ⇒ no-op. Only a strict-subset regeneration is blocked, so existing healthy backfills are unaffected.

## 6. Risks & Open Questions

- Intentional removals (a repo really was deleted, or scope legitimately narrowed) trip the guard — mitigated by `--force`, but the operator must recognize the case. Acceptable: loud-and-overridable beats silent-and-destructive.
- Same-id content changes aren't caught (only disappearance) — out of scope; a changed commit message keeps the same SHA id, and notes are already protected by PRD-0003.
- Should the live Stop-hook path default to `force` (never block an automated capture) or inherit the guard? Proposed: the hook inherits the guard but logs rather than hard-failing, since a hook can't prompt. Open for review.

## 7. Rollout & Phases

1. ✅ **Phase 1 — Guard + `--force`:** pure subset check in the handler; CLI refuses
   and reports on a would-clobber, `--force` overrides. Closes the silent-loss
   hole for git and session activity.

## 8. Future Considerations

- A `--dry-run --diff` mode that reports what a regeneration *would* change (added/dropped) without writing — useful before any backfill.
- Warn (not block) when a discovered repo's local `main` is behind its remote — addresses the upstream cause of one clobber vector (stale checkout) rather than just the symptom.
