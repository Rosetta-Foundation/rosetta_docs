# Architecture

Architecture decisions and history for the Rosetta platform.

## What is an ADR?

An **ADR** — _Architecture Decision Record_ — is a short document that captures a
single significant architectural decision: the context that forced the choice, the
decision itself, and its consequences. ADRs are immutable once accepted; when a
decision changes, a new ADR supersedes the old one rather than editing it. Together
they form a dated, append-only history of _why_ the system is the way it is.

## Status lifecycle

| Status     | Meaning                                                     |
| ---------- | ----------------------------------------------------------- |
| Proposed   | Drafted and under discussion; not yet ratified.             |
| Accepted   | Ratified; the decision is in force.                         |
| Superseded | Replaced by a later ADR (which it links to).                |
| Deprecated | No longer relevant, but retained for the historical record. |

### When to accept

An ADR flips from **Proposed** to **Accepted** when the decision has actually been
_made and committed to_ — not when the document reads well, and not when the thing
is built. Accept when all three hold:

1. **The people who own the decision have affirmatively agreed** — not "no one
   objected," but an actual decision by whoever can commit the team to the
   consequences.
2. **Every dependency that could _change_ the decision is resolved.** Distinguish
   blocking from downstream: a dependency that might alter the decision blocks
   acceptance; one that is merely later work does not.
3. **You are willing to treat it as binding** — new work must conform, and changing
   course requires a _new_ ADR that supersedes this one, not an edit.

Accepted does **not** wait for implementation, perfect wording, or unanimous
enthusiasm ("disagree and commit" still yields Accepted; record substantive dissent
in the consequences).

**Example — ADR-0002.** It stays Proposed because a blocking dependency is open: the
privacy-model review of email-grade personal stores. If that review confirms the posture
needs no change, the decision is unaltered and it can be accepted immediately; if the
review requires a different boundary for sensitive career data, that _changes_ the decision,
so it is revised while still Proposed. By contrast, ADR-0001 (philosophy, no external
gate, clearly held by the project) was Accepted from the start.

To accept: change `**Status:**` to `Accepted`, update the Records table below, and do
it in a small dedicated commit (`docs: accept ADR-NNNN`) so the ratification is itself
a reviewable, dated event.

## Records

| ADR                                                      | Title                                                          | Status   | Date       |
| -------------------------------------------------------- | -------------------------------------------------------------- | -------- | ---------- |
| [0001](ADR-0001-rosetta-philosophy.md)                   | Rosetta Philosophy                                             | Accepted | 2026-07-21 |
| [0002](ADR-0002-personal-vs-organizational-chronicle.md) | Personal Chronicle vs. Organizational Chronicle                | Proposed | 2026-07-21 |
| [0003](ADR-0003-tauri-thin-rust-core.md)                 | Tauri Apps — Thin Rust Core, Business Logic in TypeScript      | Accepted | 2026-07-24 |
| [0004](ADR-0004-shared-rosetta-core.md)                  | Shared Rosetta Core — What's Shared vs. Per-App                | Accepted | 2026-07-24 |
| [0005](ADR-0005-decentralized-by-construction.md)        | Decentralized by Construction — Git Semantics at Every Scale   | Proposed | 2026-07-31 |
| [0006](ADR-0006-ts7-bun-toolchain.md)                    | TypeScript 7 + Bun Toolchain                                   | Accepted | 2026-07-31 |
| [0007](ADR-0007-chronicle-commit-type.md)                | The `chronicle:` Commit Type — Machine-Authored Ledger Commits | Accepted | 2026-07-31 |
| [0008](ADR-0008-implementation-spec-format.md)           | Implementation Spec Format — The PRD-to-Build Contract         | Accepted | 2026-07-31 |

## Conventions

- Filename: `ADR-NNNN-short-kebab-title.md` (zero-padded, sequential).
- Each ADR opens with `**Status:**` and `**Date:**` and states one decision.
- Add every new ADR to the **Records** table above in the same change.
- All TypeScript in Rosetta follows the Handler / Service / Repository + InversifyJS
  pattern (see `../.claude/rules/architecture-hsr.md`); record deviations or
  extensions to that pattern as ADRs here.
