---
id: PRD-0004
title: Integrated Note-Taker
status: Proposed
date: 2026-07-23
owner: Russ Watson
related_adrs: [ADR-0002]
related_specs: []
supersedes: null
---

# PRD-0004: Integrated Note-Taker

> A first-class, Obsidian-style note-taking surface integrated with Rosetta, so
> capturing meeting and working notes feeds Chronicle directly instead of living
> in a separate app.

## 1. Overview & Goals

### 1.1 Purpose

[PRD-0003](PRD-0003-notes-as-authoritative-input.md) makes hand-authored notes an
authoritative input read from a per-day Markdown file. That file is editable in
any text editor — but the natural home for Markdown notes with backlinks, daily
notes, and a graph view is a tool like **Obsidian**. This capability explores a
note-taking experience integrated with Rosetta: the engineer writes notes in a
rich, linked, Markdown-native surface, and those notes flow into Chronicle as
authoritative input with zero copy-paste.

PRD-0003 has shipped, so the authoritative notes file now exists and is
git-tracked. This capability starts with the cheapest possible integration —
**point an Obsidian vault at the `chronicles/notes/` directory** — validating the
"write in a rich editor, notes flow to Chronicle" workflow with zero code and no
proprietary lock-in, before deciding whether a plugin or native surface is worth
building.

### 1.2 Goals

- Provide a Markdown-native note-taking surface (Obsidian-compatible) whose daily notes are Chronicle's authoritative notes input.
- Preserve the PRD-0003 guarantee: notes are input, never regenerated or clobbered.
- Support backlinks / wikilinks between notes, and from notes to Chronicle entities (repos, PRs, tickets, sessions).
- Keep everything local, Markdown-on-disk, and git-tracked — no proprietary lock-in.

### 1.3 Non-Goals

- Not building a note-taking app from scratch if an existing tool (Obsidian + its vault) can be pointed at the Chronicle notes directory.
- Not replacing PRD-0003's plain-file capture — this is a richer surface *over* the same authoritative files.
- Not organizational publication — notes stay private (ADR-0002).

### 1.4 Acceptance Criteria

**Phase 1 (vault-over-directory):**

- [x] A documented setup points an Obsidian vault at the Chronicle repo's `chronicles/notes/` directory, with the daily-notes plugin configured to write `YYYY-MM-DD.md` into it.
- [x] A note written in Obsidian for a given day is picked up by the next Chronicle generation for that day with no export step — because the file *is* the PRD-0003 authoritative `chronicles/notes/<date>.md`.
- [x] A daily-note template seeds the file in the format `NotesRepository` already parses (bullet lines, optional `[HH:MM]`), so vault-authored notes need no reformatting.
- [x] The workflow is documented well enough that setup is a copy-paste, not a research task.

**Later phases (deferred — see Rollout):**

- [ ] Wikilinks between notes resolve to Chronicle entities (repos, PRs, tickets, sessions).

## 2. Users & Motivation

Serves the engineer who already thinks in Markdown and linked notes: one place to
write, automatically feeding durable organizational memory. Removes the
app-switching and copy-paste tax between "where I take notes" and "where notes
become knowledge."

## 3. Approach

Three candidate directions, cheapest first. **Phase 1 commits to vault-over-directory;**
the other two remain deferred until daily use justifies the investment.

- **Vault-over-directory (Phase 1):** point an Obsidian vault at the Chronicle
  repo's `chronicles/notes/` directory so daily notes *are* the authoritative
  files. Obsidian edits Markdown in place; Chronicle reads the same files. No
  code, no sync, no lock-in — it leans entirely on PRD-0003's file contract. The
  daily-notes plugin is configured to name files `YYYY-MM-DD.md` and seed them
  from a template matching the format `NotesRepository` parses. Delivered as a
  setup guide in this repo, not as code in Chronicle.
- **Plugin/bridge (deferred):** an Obsidian plugin (or CLI sync) that maps vault
  daily notes ↔ Chronicle notes and resolves wikilinks to Chronicle entities.
  Only worth building if vault-over-directory proves too limited (e.g. links
  don't resolve, or the notes dir shouldn't be the vault root).
- **Native surface (deferred):** a Rosetta-owned editor (e.g. in Wayfinder) —
  highest effort, most control, least reliance on a third-party app.

## 4. Data Contracts

Reuses the PRD-0003 notes file contract (`chronicles/notes/<date>.md`) and the
`INotesStore` / `INotesRepository` interfaces. Any entity-linking would extend
Chronicle's evidence/entity model — TBD.

Phase 1 setup guide: [`docs/obsidian-notes-setup.md`](../docs/obsidian-notes-setup.md).

## 5. Constraints & Dependencies

- **Depends on:** PRD-0003 (authoritative notes input) as the foundation.
- **Relates to:** Wayfinder as a potential host for a native surface and for
  resolving note ↔ entity links.
- **Constraint:** Markdown-on-disk, local, git-tracked — no proprietary format.

## 6. Risks & Open Questions

- Buy vs. build: point Obsidian at the vault vs. a bridge plugin vs. a native editor — which balances effort and integration?
- Wikilink semantics: how do note links resolve to Chronicle entities (repos, PRs, tickets, sessions)?
- Sync model: is the tool editing the files in place, or syncing into them (and how are conflicts handled)?
- Does this belong in Chronicle, Wayfinder, or as a standalone integration?

## 7. Rollout & Phases

1. ✅ **Phase 1 — Vault-over-directory (this PRD):** a setup guide + daily-note
   template that make `chronicles/notes/` an Obsidian vault. Zero code in
   Chronicle; validates the workflow. Delivered as `docs/obsidian-notes-setup.md`
   in this repo.
2. **Phase 2 — Bridge / links (deferred):** only if Phase 1 proves too limited.
   A plugin or CLI that resolves note↔entity wikilinks.
3. **Phase 3 — Native surface (deferred):** a Rosetta/Wayfinder-owned editor.

## 8. Future Considerations

- Bi-directional links between notes and Chronicle entities feeding Wayfinder queries.
- Templated daily notes pre-populated with the day's meetings/activity.
- Mobile/quick-capture into the same authoritative files.
