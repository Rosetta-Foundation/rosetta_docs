---
id: PRD-0003
title: Meeting Notes as Authoritative Input
status: Shipped
date: 2026-07-23
owner: Russ Watson
related_adrs: [ADR-0002]
related_specs: []
supersedes: null
---

# PRD-0003: Meeting Notes as Authoritative Input

> Treat human-authored meeting notes as durable, first-class **input** — never
> regenerated, never clobbered — sourced from a file the engineer owns rather
> than scraped back out of rendered Markdown.

## 1. Overview & Goals

### 1.1 Purpose

Git commits and Claude sessions are _derived_ activity: Chronicle can always
rebuild them from `git log` or the transcripts. Meeting notes are the opposite —
they are **primary data authored by a human and exist nowhere else**. Today they
only survive a regeneration because `parseExistingNotes` scrapes them back out of
the rendered `## Notes & Discussions` section with a regex. That round-trip is
lossy (structure is flattened to one line) and unsafe (a render-format change,
heading rename, or unmatched bullet silently destroys the note, with no source to
recover it from).

This capability makes notes an authoritative **input tier**: the engineer writes
them into a file they own, Chronicle reads that file as the source of truth, and
the rendered section becomes pure output. Regeneration can never clobber a note
because regeneration never writes one.

### 1.2 Goals

- Persist human-authored notes as a durable, git-tracked artifact the engineer edits directly.
- Make `NotesRepository` read notes from that artifact, not from rendered Chronicle Markdown.
- Guarantee regeneration is non-destructive: notes are read-only input to synthesis.
- Support two capture paths that merge cleanly: hand-typed daily notes and tool-ingested notes.
- Preserve idempotent append-as-you-go via the existing content-hash dedup.

### 1.3 Non-Goals

- Not regenerating, summarizing, or rewriting note content — notes are authoritative as written.
- Not building every ingest adapter now (calendar/Granola/Otter/Slack) — Phase 2 adds the first; the rest follow the source pattern.
- Not organizational publication — notes stay in the private Chronicle (ADR-0002); they may name people and must be re-sanitized on any publication.
- Not a notes editing UI — the artifact is a plain Markdown file.

### 1.4 Acceptance Criteria

- [x] A per-day human-owned notes file exists at a stable path (e.g. `chronicles/notes/<date>.md`), git-tracked and safe to hand-edit.
- [x] `NotesRepository.getActivity` reads notes from that file (and any inline `notes` input), never from rendered Chronicle Markdown.
- [x] Regenerating a Chronicle for a day with existing notes never drops, reorders, or flattens a note beyond its authored form.
- [x] `parseExistingNotes` is removed from the critical path for preserving notes across regenerations.
- [x] Re-running over the same day never duplicates a note (content-hash stable id, as today).
- [x] Hand-typed notes and ingested notes for the same day merge, with identical entries deduplicated.
- [x] Deleting the rendered Chronicle and regenerating restores the notes section verbatim from the notes file.

## 2. Users & Motivation

Serves the engineer first: a place to jot standup / 1:1 / design-review notes
that becomes durable knowledge automatically, with zero risk that tomorrow's
regeneration erases today's note. Later serves managers and agents — meeting
context ("what was decided, with whom") is exactly the durable "why" Chronicle
exists to capture. Removes a real data-loss hazard in the current design.

## 3. Approach

Notes are an **input tier**, distinct from the regenerable git/Claude sidecars in
[PRD-0002](PRD-0002-multi-repo-git-and-synthesis.md): those are caches (rebuild
from source if deleted); notes are authoritative (deletion is permanent), so the
artifact is the medium humans edit — Markdown, not machine JSON.

|            | Git / Claude sidecars (PRD-0002) | Notes artifact (this PRD)         |
| ---------- | -------------------------------- | --------------------------------- |
| Origin     | derived from a source            | authored by a human               |
| If deleted | regenerate from source           | data lost forever                 |
| Treatment  | cache — safe to rebuild          | authoritative — never regenerated |
| Format     | structured JSON                  | human-editable Markdown           |

Two capture paths, merged by the existing dedup:

1. **Hand-typed** — the engineer edits `chronicles/notes/<date>.md` directly.
2. **Ingested** — a source adapter pulls notes from an external tool and normalizes them.

`NotesRepository` reads both, dedupes by content-hash id, and emits `Activity[]`.
The rendered `## Notes & Discussions` section is pure output.

## 4. Data Contracts

```ts
// Reads authoritative notes for a day from the human-owned file (+ inline input),
// never from rendered Chronicle Markdown.
export interface INotesRepository {
  getActivity(window: ChronicleWindow, notes?: string): Promise<Activity[]>;
}

// New: locate/read/write the per-day human-owned notes file.
export interface INotesStore {
  readDaily(repoPath: string, date: string): Promise<string | null>;
  /** Append-safe write for live capture; dedups by content-hash id. */
  appendDaily(repoPath: string, date: string, entry: string): Promise<void>;
}
```

Reuses existing `Activity` / `Evidence` / `ChronicleWindow` types (`src/types.ts`)
and the content-hash `noteId` dedup already in `NotesRepository`.

## 5. Constraints & Dependencies

- **Architecture:** Handler / Service / Repository + InversifyJS. The notes file
  is resource access (a repository); merging/dedup is orchestration.
- **Privacy (ADR-0002):** notes are private; local, git-tracked destination.
- **Depends on / relates to:** PRD-0002's synthesis direction (retiring the lossy
  Markdown re-parse) — this PRD does the same for notes and closes a data-loss hole.
- **Backward compatibility:** on first run for a pre-feature day, migrate notes
  once from the rendered Markdown into the notes file, then treat the file as truth.

## 6. Risks & Open Questions

- One file per day vs. one per meeting — per-day is simplest; per-meeting adds structure but more files.
- Structured note fields (attendees, decisions, action items) vs. free-form lines — start free-form, structure later.
- Ingest source for Phase 2 — calendar chosen (shipped). Granola/Otter/Slack remain open.
- Conflict handling if a note is edited in the file _and_ re-ingested from a tool — dedup by content-hash, but near-duplicates (reworded) may slip through.
- Migration correctness for existing days whose only copy of a note is the rendered Markdown.

## 7. Rollout & Phases

1. ✅ **Phase 1 — Hand-typed authoritative notes:** add the per-day notes file and
   `NotesStore`; point `NotesRepository` at it; make the rendered section pure
   output; retire `parseExistingNotes` from the preservation path; one-time
   migration from existing rendered notes. Closes the clobbering/data-loss hole.
2. **Phase 2 — First ingest adapter (calendar):** ✅ _Shipped 2026-07-23 (PR #23)._
   `CalendarRepository` reads a `.ics` export (RFC 5545, no OAuth, works with any
   provider — Google, Outlook, Apple), emits one `Activity` per in-window meeting
   (title, time, attendees). Merged via content-hash dedup alongside hand-typed
   notes. CLI gains `--calendar <path>` / `$CHRONICLE_CALENDAR` env var, threaded
   into both backfill and append-session. 20 tests added: `ics.utils`, repository
   filtering, and service DI binding. Richer content sources (Granola/Otter
   transcripts, Slack) follow the same source-repository pattern.

## 8. Future Considerations

- Additional ingest adapters (calendar, Granola, Otter, Slack) as new repositories.
- Live append-as-you-go: write notes to the file as meetings happen (Stop-hook / integration), aligning with the live-sourcing direction.
- Structured note schema (attendees, decisions, action items) feeding Wayfinder "what was decided, with whom" queries.
