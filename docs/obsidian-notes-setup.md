# Obsidian Notes Setup (PRD-0004 Phase 1)

Point Obsidian at your Chronicle's notes directory so meeting and working notes
you type in Obsidian flow into the Daily Chronicle automatically — no export, no
copy-paste. The Markdown file Obsidian edits *is* the authoritative notes input
Chronicle reads (PRD-0003).

> **Why this works with zero code:** Chronicle reads notes from
> `<your-chronicle-repo>/chronicles/notes/<YYYY-MM-DD>.md`. Obsidian edits
> Markdown files in place. Make Obsidian's daily note write to that path and the
> two are the same file.

## One-time setup

1. **Open the notes directory as a vault.** In Obsidian → *Open folder as vault*,
   choose your personal Chronicle repo's notes directory:

   ```
   <CHRONICLE_REPO>/chronicles/notes
   ```

   (e.g. `~/projects/rosetta/rosetta_chronicle_<you>/chronicles/notes`). Opening
   the `notes/` subfolder — not the repo root — keeps the vault focused on notes
   and avoids Obsidian writing `.obsidian/` config among your chronicle files.

2. **Enable the Daily Notes core plugin.** Settings → *Core plugins* → toggle
   **Daily notes** on.

3. **Configure Daily Notes** (Settings → *Daily notes*):
   - **Date format:** `YYYY-MM-DD` — this must match exactly; it's the filename
     Chronicle looks for.
   - **New file location:** the vault root (i.e. `chronicles/notes/` itself).
   - **Template file location:** point at `_daily-template.md` (created below).

4. **Add the daily-note template.** Create `_daily-template.md` in the vault with
   the contents in the next section. (The leading underscore keeps it sorting
   above the dated notes; Chronicle ignores it — only `YYYY-MM-DD.md` files are
   read for a given day.)

That's it. Press the Daily Note button (or <kbd>Cmd/Ctrl-P</kbd> → *Open today's
daily note*) and start typing. The next `chronicle backfill` / live capture for
that day picks the notes up.

## Daily-note template

Save as `_daily-template.md` in the vault. The bullet + optional `[HH:MM]` format
is exactly what Chronicle's `NotesRepository` parses, so nothing needs
reformatting:

```markdown
# {{date:YYYY-MM-DD}}

## Notes

- [{{time:HH:mm}}]

## Meetings

-

## Decisions

-
```

Notes to keep in mind:

- **One bullet per note.** Each `- ...` line becomes one Chronicle activity.
- **`[HH:MM]` is optional.** A leading `[14:32]` on a bullet anchors that note to
  a time; without it the note anchors to the start of the day. The
  `{{time:HH:mm}}` token pre-fills the current time when you create the note —
  delete it if a note isn't time-specific.
- **Headings and empty bullets are ignored.** `## Notes`, `## Meetings`, blank
  lines, and a bare `-` don't become activities — only bullets with text do. So
  the template's structure is for *your* organization; Chronicle just harvests
  the non-empty bullets.
- **Bold/links/sub-text are fine.** The note text is taken verbatim after the
  bullet, so `- discussed **Okta** rollout with [[Vinay]]` is captured as-is.

## How it flows into the Chronicle

1. You type notes in Obsidian → saved to `chronicles/notes/<date>.md`.
2. `chronicle backfill` (or the live Stop-hook) runs for that day.
3. `NotesRepository` reads the file, turns each non-empty bullet into an
   `Activity`, and the Daily Chronicle renders a **Notes & Discussions** section.
4. Because notes are **authoritative input** (PRD-0003), regeneration never
   rewrites your file — Chronicle only ever reads it. Edit freely in Obsidian;
   your words are the source of truth.

## Gotchas

- **Date format must be `YYYY-MM-DD`.** Any other Obsidian date format writes a
  filename Chronicle won't find for that day.
- **Commit your notes.** The notes file is git-tracked in your Chronicle repo; a
  `chronicle` run commits it alongside the rendered doc, but if you want notes
  preserved before the next run, commit them yourself.
- **Wikilinks don't resolve to Chronicle entities yet.** `[[Vinay]]` is captured
  as literal text today. Resolving links to repos/PRs/tickets/sessions is
  deferred to PRD-0004 Phase 2 — it isn't part of this setup.
- **Vault config files.** Obsidian writes an `.obsidian/` directory in the vault.
  It's harmless (Chronicle ignores it), but you may want to add `.obsidian/` to
  the Chronicle repo's `.gitignore` if you'd rather not track editor state.
