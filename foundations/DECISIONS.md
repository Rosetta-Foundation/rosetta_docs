# Decisions

**Status:** Founding Context (v0.1)

**Date:** 2026-07-31

The long-lived architectural and philosophical decisions, in one place.
Detailed records live as ADRs in [`architecture/`](../architecture/); this
file is the constitution-level index — what is already settled, so it is not
relitigated by accident. Add a row when a decision is meant to bind for years,
not sprints.

---

## Philosophical

| Decision                                          | Meaning                                                                                                                              | Record                                                                                                                                          |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| Context is the product                            | Documentation is a byproduct; Rosetta preserves the why, not just the what.                                                          | [ADR-0001](../architecture/ADR-0001-rosetta-philosophy.md)                                                                                      |
| Engineering is the first domain, not the boundary | Rosetta is infrastructure for collective intelligence; the machinery is domain-agnostic. Widening scope is not a change of identity. | [CONTEXT.md](CONTEXT.md) (2026-07-31)                                                                                                           |
| The hero is humanity                              | Rosetta is a tool; success is measured by what people preserve and build together, never engagement.                                 | [CONTEXT.md](CONTEXT.md)                                                                                                                        |
| Private by default, shared by intention           | Knowledge is born personal; sharing is deliberate. Promotion is the only path outward.                                               | [ADR-0002](../architecture/ADR-0002-personal-vs-organizational-chronicle.md), [PRD-0006](../product/PRD-0006-artifact-capture-and-promotion.md) |
| Trust is provenance, not position                 | Authority comes from evidence and attribution carried in the data, never from which server holds it.                                 | [ADR-0001](../architecture/ADR-0001-rosetta-philosophy.md), [ADR-0005](../architecture/ADR-0005-decentralized-by-construction.md)               |

## Architectural

| Decision                                                                          | Meaning                                                                                                                                                                                   | Record                                                                                                                                             |
| --------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Personal and organizational chronicles are separate repos over one neutral engine | The engine has no home server; data and machinery never fuse.                                                                                                                             | [ADR-0002](../architecture/ADR-0002-personal-vs-organizational-chronicle.md)                                                                       |
| Decentralized by construction                                                     | Same machinery at every scale; no tier requires a central server; services are conveniences, never dependencies. Every future design must pass the no-server / N=2 / same-code-path test. | [ADR-0005](../architecture/ADR-0005-decentralized-by-construction.md)                                                                              |
| The personal chronicle belongs to the person                                      | Provisioned in the user's own account, not any org. Leaving an org must never cost you your memory.                                                                                       | [ADR-0005](../architecture/ADR-0005-decentralized-by-construction.md) exit test                                                                    |
| Open boundaries: schema-first ingestion, pluggable inference                      | Producers push schema-valid envelopes; inference backends are configuration. Both boundaries outlive any vendor.                                                                          | [PRD-0015](../product/PRD-0015-chronicle-activity-schema-and-open-ingestion.md), [PRD-0016](../product/PRD-0016-offline-on-device-intelligence.md) |
| Thin Rust core; business logic in TypeScript                                      | Tauri apps keep the native shell minimal.                                                                                                                                                 | [ADR-0003](../architecture/ADR-0003-tauri-thin-rust-core.md)                                                                                       |
| One shared core, per-app surfaces                                                 | Contracts and pure logic are shared machinery; apps are consumers.                                                                                                                        | [ADR-0004](../architecture/ADR-0004-shared-rosetta-core.md)                                                                                        |
| Handler / Service / Repository + DI everywhere                                    | Mandatory layering for all TypeScript in every repo.                                                                                                                                      | `.claude/rules/architecture-hsr.md`                                                                                                                |
| Machine-authored ledger commits are first-class                                   | `chronicle:` commit type + provenance trailers; the machinery never chronicles itself.                                                                                                    | [ADR-0007](../architecture/ADR-0007-chronicle-commit-type.md)                                                                                      |

## Operational

| Decision                                     | Meaning                                                                                         | Record                                                                       |
| -------------------------------------------- | ----------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| Everything is legible to humans and machines | Structured frontmatter, evidence links, parseable conventions — both audiences are first-class. | [ADR-0001](../architecture/ADR-0001-rosetta-philosophy.md), repo conventions |
| Toolchain choices are recorded, not accreted | Significant tooling shifts get an ADR.                                                          | [ADR-0006](../architecture/ADR-0006-ts7-bun-toolchain.md)                    |
