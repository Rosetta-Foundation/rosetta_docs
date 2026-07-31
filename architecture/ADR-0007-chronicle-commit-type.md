# ADR-0007: The `chronicle:` Commit Type — Machine-Authored Ledger Commits

**Status:** Accepted

**Date:** 2026-07-31

---

> Commits written _by_ Chronicle machinery into ledger repositories carry the
> dedicated Conventional Commit type `chronicle:`, with the ledger operation as
> scope and machine-readable provenance in git trailers. Activity collection
> never ingests `chronicle:` commits — self-exclusion is protocol, not
> path-matching.

---

# Background

Chronicle's engine persists its daily output by committing to the personal
chronicle data repository. Today those commits are labeled
`chore: daily chronicle <date>` — and the engine then has to special-case
_excluding its own output repo by path_ during activity collection, so its own
ledger writes don't pollute the next day's "Work completed" section.

Two problems:

1. **`chore` is a lie.** Conventional Commit _types_ describe the nature of a
   change; _scopes_ describe the target. A ledger append is not maintenance on
   a codebase — it is the system writing its memory. The ledger is a
   first-class artifact class ([ADR-0002](ADR-0002-personal-vs-organizational-chronicle.md));
   it deserves a first-class type.
2. **Path-based self-exclusion is fragile.** It works only while ledger and
   code live in different repositories, and it encodes an incidental fact
   (where the output repo happens to be) instead of the actual rule (Chronicle
   must not chronicle itself).

The open-ingestion work
([PRD-0015](../product/PRD-0015-chronicle-activity-schema-and-open-ingestion.md))
and promotion pipeline
([PRD-0006](../product/PRD-0006-artifact-capture-and-promotion.md),
[PRD-0009](../product/PRD-0009-coherence-protocol.md)) will add more
machine-authored commit kinds, and [ADR-0005](ADR-0005-decentralized-by-construction.md)
makes commit-level provenance the trust mechanism between peers. The convention
needs to be settled before those land.

# Decision

## 1. `chronicle:` marks machine authorship, not subject matter

The type is reserved for commits **written by Chronicle machinery into ledger
(data) repositories**. Commits _touching_ Chronicle-the-product's code use
normal types — the repository boundary already identifies the product, and
`feat(chronicle): ...` remains available as an ordinary scope where useful.

## 2. The scope names the ledger operation

| Commit prefix         | Operation                                               |
| --------------------- | ------------------------------------------------------- |
| `chronicle(daily):`   | Daily chronicle synthesis (render + sidecar + notes)    |
| `chronicle(queue):`   | Queue item lifecycle writes                             |
| `chronicle(notes):`   | Authoritative notes file appends                        |
| `chronicle(ingest):`  | Inbox appends from external producers (PRD-0015)        |
| `chronicle(promote):` | Promotions landing in an org repo (PRD-0006 / PRD-0009) |

The subject after the colon is the operation's natural key — for
`chronicle(daily):` it is the window, e.g. `chronicle(daily): 2026-07-31`.

## 3. Trailers carry machine-readable provenance

The type says _what kind_; [git trailers](https://git-scm.com/docs/git-interpret-trailers)
say _exactly what_. Git parses trailers natively, so the commit history becomes
a queryable structured ledger — ADR-0005's "trust is provenance, not position"
made concrete at the commit level.

| Trailer             | Meaning                                      | Example                   |
| ------------------- | -------------------------------------------- | ------------------------- |
| `Chronicle-Window:` | The window the commit's artifacts cover      | `2026-07-31`              |
| `Generated-By:`     | Producer name@version                        | `chronicle@0.1.0`         |
| `Chronicle-Schema:` | Schema version of ingested payloads (ingest) | `activity.v1`             |
| `Model-Provenance:` | Backend/model that synthesized (PRD-0016)    | `llama-cpp/llama-3.3-70b` |

`Chronicle-Window:` and `Generated-By:` are mandatory on every `chronicle:`
commit. `Chronicle-Schema:` is mandatory on `chronicle(ingest):`.
`Model-Provenance:` becomes mandatory once PRD-0016's inference abstraction
lands. Example:

```
chronicle(daily): 2026-07-31

Chronicle-Window: 2026-07-31
Generated-By: chronicle@0.1.0
```

## 4. Self-exclusion is protocol

Activity collection **never ingests commits whose Conventional Commit type is
`chronicle`**. The rule replaces path identity as the semantic guard: it works
when ledger and code share a repository, when a discovered workspace contains
someone else's chronicle repo, and at every federation tier. Path-based
exclusion of the output repo may remain as an optimization, but the protocol
rule is the invariant.

## 5. No `wayfinder:` type — types track the machinery

Wayfinder's invisible-git writes use the same engine and ledger format
([ADR-0004](ADR-0004-shared-rosetta-core.md)), so they emit `chronicle:` too.
Minting a type per surface would grow forever; the machinery is the author, the
app is just the surface.

## 6. Human edits to ledger content use normal types

A person hand-correcting a note in their personal repo commits
`fix: correct meeting note`, not `chronicle(manual):`. The type marks machine
authorship; human edits are honestly different — and remain visible to
activity collection, which is correct (a human editing their chronicle _is_
engineering activity only when it happens in a code repo; ledger repos are
excluded by the path optimization as today).

# Consequences

**Positive:**

- Ledger history is self-describing and queryable
  (`git log --grep='^chronicle(' `, `git interpret-trailers`).
- Self-exclusion stops depending on repository layout.
- The provenance trailer schema gives PRD-0015 envelopes and PRD-0009
  coherence gates a commit-level anchor, per ADR-0005.
- Enforcement is one regex change in the shared `commit-msg` hook template.

**Negative / costs:**

- Nonstandard type: external Conventional Commits tooling (changelog
  generators, commitlint presets) needs the type added to its allowlist.
  Ledger repos rarely run such tooling; code repos never emit the type.
- Existing ledger history keeps `chore: daily chronicle` commits; the
  self-exclusion protocol therefore also matches that legacy subject during a
  deprecation window (the path optimization already covers them in practice).

# Adoption

1. **dev-scripts** — add `chronicle` to the `commit-msg` hook template's types
   regex and to the workspace `CLAUDE.md` valid-types list.
2. **chronicle** — engine emits `chronicle(daily): <window>` with
   `Chronicle-Window:` + `Generated-By:` trailers; activity collection applies
   the type-based self-exclusion rule.
3. **wayfinder** — adopts the same emission when its ledger writes land
   (no change today).
