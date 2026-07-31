# Glossary

**Status:** Founding Context (v0.1)

**Date:** 2026-07-31

Canonical terminology. When a term here conflicts with usage elsewhere, this
file wins — update the other document.

---

## Platform

**Rosetta** — the platform: infrastructure for collective intelligence, a
shared memory layer for people, projects, and AI. Named for the Rosetta Stone:
it does not invent knowledge, it reveals relationships between representations
of it.

**Chronicle** (capitalized) — the memory engine. Transforms activity into
durable, structured knowledge. Presentation-agnostic; everything else consumes
it. Repo: `rosetta_chronicle`.

**Wayfinder** — the knowledge guide. A local-first desktop app that consumes
Chronicle to help people navigate, ask, and understand. Repo:
`rosetta_wayfinder`.

**Civic Blueprint** — an anticipated application built on Rosetta
infrastructure, oriented at civic coordination. Not part of the platform;
named in founding context as proof that Rosetta must stay domain-agnostic.

---

## Knowledge Objects

**chronicle** (lowercase) — a data repository holding someone's (or some
group's) knowledge ledger, operated on by the engine. Personal, team, org,
community, and global chronicles are the same machinery with different policy
(ADR-0002, ADR-0005).

**ledger** — the append-mostly record inside a chronicle repo: daily
chronicles, notes, queue state, sidecars. Written by machinery via
`chronicle:` commits (ADR-0007).

**daily chronicle** — the synthesized document for one day's window: rendered
Markdown plus its structured sidecar.

**sidecar** — the structured JSON source of truth accompanying a rendered
document (`chronicles/.data/<date>.json`).

**activity** — one unit of captured work (a commit, an agent session, a note,
a meeting) with evidence attached.

**evidence** — the traceable reference an activity carries (SHA, session id,
note ref). The unit of provenance (ADR-0001).

**envelope** — the wire format for pushed activity: schema version + producer
provenance + the activity itself (PRD-0015).

**artifact** — a captured content object (document, decision, book copy)
stored in a chronicle and eligible for promotion (PRD-0006).

---

## Processes

**synthesis** — composing raw activity into knowledge (daily chronicles,
titles, tags) via an inference backend (PRD-0016).

**ingestion** — accepting activity from producers: built-in source adapters or
schema-validated push from external producers (PRD-0015).

**promotion** — the deliberate, gated act of moving knowledge from a personal
chronicle to a shared one — the "shared by intention" mechanism and the
compliance boundary (PRD-0006).

**coherence protocol** — the gate that keeps a shared chronicle
self-consistent as promotions land: contradiction checks, review, merge
(PRD-0009).

**capture tier** — a device class that only captures and queues envelopes
(wearables, rigs, robots), deferring synthesis to where compute lives
(PRD-0016).

**producer** — anything that emits activity into a chronicle. External
producers use `ext:<namespace>/<name>` sources and speak the schema; the
hardware is never ours (PRD-0015).

---

## Conventions

**`chronicle:` commit type** — the Conventional Commit type reserved for
machine-authored ledger commits, e.g. `chronicle(daily): 2026-07-31`, with
provenance trailers (`Chronicle-Window:`, `Generated-By:`). Never hand-written
by humans; never ingested as activity (ADR-0007).

**HSR** — Handler / Service / Repository: the mandatory TypeScript layering in
all Rosetta repos, with InversifyJS dependency injection.

**PRD / ADR** — Product Requirements Document (capability framed before it is
built, in `product/`) / Architecture Decision Record (decision already made,
in `architecture/`).

**implementation spec** — the phased spec derived from an accepted PRD that
scopes one shippable phase: engineering tasks, dependencies, and acceptance
criteria. The contract between the PRD ("what and why") and the build
(PRD → phased implementation specs → build).

**stewardship** — the founding philosophy: ownership, influence, and
technology are temporary; the obligation to leave shared knowledge stronger
than you found it endures.
