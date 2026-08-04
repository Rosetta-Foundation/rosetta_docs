# Product

Product Requirements Documents (PRDs) for the Rosetta platform.

## What is a PRD?

A **PRD** — _Product Requirements Document_ — frames a capability **before** it is
built: its purpose, goals, non-goals, and acceptance criteria. Where an
[ADR](../architecture) records a decision already made, a PRD defines what to
build and what "done" means. A PRD is the input to phased implementation specs
(PRD → phased implementation specs → build).

PRDs are authored to be legible to **both humans and machines**: structured
frontmatter, list-based goals/non-goals, checkbox acceptance criteria, and
explicit links. The objective is that agents can author, consume, and act on
them — not only people.

## Status lifecycle

| Status     | Meaning                                                 |
| ---------- | ------------------------------------------------------- |
| Draft      | Being written; not yet circulated.                      |
| Proposed   | Circulated and under review; not yet committed to.      |
| Accepted   | Committed to; work may proceed against it.              |
| Shipped    | Planned phases delivered; acceptance criteria met.      |
| Superseded | Replaced by a later PRD (which it links to).            |
| Deprecated | No longer relevant, retained for the historical record. |

**Where to look:** **In flight** is the open backlog (Draft / Proposed / Accepted
with remaining phases). **Shipped** is complete delivery. **Retired** is
Superseded / Deprecated. Paths stay under this folder — status + index
grouping, not an archive directory.

## In flight

Open PRDs — available for review, commitment, or remaining phase work.

| PRD                                                              | Title                                         | Status   | Phases              | Date       |
| ---------------------------------------------------------------- | --------------------------------------------- | -------- | ------------------- | ---------- |
| [0004](PRD-0004-integrated-note-taker.md)                        | Integrated Note-Taker                         | Proposed | ✅ 1 ⬜ 2 ⬜ 3      | 2026-07-23 |
| [0006](PRD-0006-artifact-capture-and-promotion.md)               | Artifact Capture & Promotion to Org Knowledge | Proposed | ⬜ 1 ⬜ 2 ⬜ 3      | 2026-07-24 |
| [0007](PRD-0007-personal-work-queue.md)                          | Personal Work Queue                           | Accepted | ✅ 1 ⬜ 2 ⬜ 3      | 2026-07-24 |
| [0009](PRD-0009-coherence-protocol.md)                           | Org Knowledge Coherence Protocol              | Proposed | ⬜ 1 ⬜ 2 ⬜ 3      | 2026-07-24 |
| [0010](PRD-0010-wayfinder-local-app.md)                          | Wayfinder Local App                           | Proposed | ⬜ 1 ⬜ 2 ⬜ 3      | 2026-07-24 |
| [0012](PRD-0012-wayfinder-ai-workspace.md)                       | Wayfinder AI Workspace                        | Proposed | ⬜ 1 ⬜ 2 ⬜ 3      | 2026-07-26 |
| [0013](PRD-0013-markdown-rendering-in-chat.md)                   | Markdown Rendering in Ask Wayfinder Responses | Proposed | ⬜ 1                | 2026-07-27 |
| [0014](PRD-0014-light-dark-theme.md)                             | Light & Dark Theme                            | Proposed | ⬜ 1 ⬜ 2           | 2026-07-27 |
| [0015](PRD-0015-chronicle-activity-schema-and-open-ingestion.md) | Chronicle Activity Schema & Open Ingestion    | Draft    | ⬜ 1 ⬜ 2 ⬜ 3      | 2026-07-31 |
| [0016](PRD-0016-offline-on-device-intelligence.md)               | Offline On-Device Intelligence                | Draft    | ⬜ 1 ⬜ 2 ⬜ 3 ⬜ 4 | 2026-07-31 |
| [0017](PRD-0017-wayfinder-voice-audio-surface.md)                | Wayfinder Voice & Audio Surface               | Draft    | ⬜ 1 ⬜ 2 ⬜ 3      | 2026-07-31 |
| [0018](PRD-0018-sdlc-event-daemon.md)                            | SDLC Event Daemon                             | Draft    | ⬜ 1 ⬜ 2 ⬜ 3      | 2026-08-04 |
| [0019](PRD-0019-self-healing-run-engine.md)                      | Self-Healing Run Engine                       | Draft    | ⬜ 1 ⬜ 2 ⬜ 3      | 2026-08-04 |
| [0020](PRD-0020-delivery-truth-and-run-observability.md)         | Delivery Truth & Run Observability            | Draft    | ⬜ 1 ⬜ 2 ⬜ 3      | 2026-08-04 |
| [0021](PRD-0021-docs-closeout-as-run-step.md)                    | Docs Closeout as a Run Step                   | Draft    | ⬜ 1 ⬜ 2 ⬜ 3      | 2026-08-04 |
| [0022](PRD-0022-planning-side-role-skills.md)                    | Planning-Side Role Skills                     | Draft    | ⬜ 1 ⬜ 2 ⬜ 3      | 2026-08-04 |

## Shipped

Planned phases complete; acceptance criteria met. Kept in place as product memory.

| PRD                                              | Title                                               | Status   | Phases                   | Date       |
| ------------------------------------------------ | --------------------------------------------------- | -------- | ------------------------ | ---------- |
| [0001](PRD-0001-claude-code-source.md)           | Claude Code Source                                  | Shipped  | ✅ 1 ✅ 2 ✅ 3           | 2026-07-22 |
| [0002](PRD-0002-multi-repo-git-and-synthesis.md) | Multi-Repo Git Discovery & Two-Tier Daily Synthesis | Shipped  | ✅ 1 ✅ 2                | 2026-07-23 |
| [0003](PRD-0003-notes-as-authoritative-input.md) | Meeting Notes as Authoritative Input                | Shipped  | ✅ 1 ✅ 2                | 2026-07-23 |
| [0005](PRD-0005-clobber-guard.md)                | Regeneration Clobber Guard                          | Shipped  | ✅ 1                     | 2026-07-23 |
| [0008](PRD-0008-local-workspace-folders.md)      | Local Workspace Folders                             | Shipped  | ✅ 1                     | 2026-07-24 |
| [0011](PRD-0011-full-loop-sdlc-automation.md)    | Full-Loop SDLC Automation                           | Accepted | ✅ 1 ✅ 2 ✅ 3 · 4 Draft | 2026-07-25 |

## Retired

Superseded or Deprecated PRDs (none yet).

## Conventions

- Filename: `PRD-NNNN-short-kebab-title.md` (zero-padded, sequential).
- Start from [`TEMPLATE.md`](TEMPLATE.md); keep the frontmatter fields.
- Add every new PRD to the **In flight** table in the same change.
- When all planned phases are ✅ and acceptance criteria are met, flip
  `status: Shipped` and move the row to **Shipped**.
- Move Superseded / Deprecated rows to **Retired** (do not relocate files).
- Link related ADRs and, once implementation is sliced, the phase specs.
- Implementation specs are sliced from accepted PRDs — one per rollout phase,
  starting from [`SPEC-TEMPLATE.md`](SPEC-TEMPLATE.md) and committed to the
  **target repo** as `specs/<PRD-ID>/phase-<n>-spec.md` (see
  [ADR-0008](../architecture/ADR-0008-implementation-spec-format.md)).
