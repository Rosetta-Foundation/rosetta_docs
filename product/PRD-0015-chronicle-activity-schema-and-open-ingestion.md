---
id: PRD-0015
title: Chronicle Activity Schema & Open Ingestion
status: Draft
date: 2026-07-31
owner: Russ Watson
related_adrs: [ADR-0001, ADR-0002, ADR-0004, ADR-0005]
related_specs: []
supersedes: null
---

# PRD-0015: Chronicle Activity Schema & Open Ingestion

> A formal, versioned schema for Chronicle activity and a push-based ingestion
> boundary so **any** producer — scripts, agents, MCP servers, devices we
> haven't imagined — can contribute activity without Chronicle shipping an
> adapter for it.

## 1. Overview & Goals

### 1.1 Purpose

Chronicle today is pull-only and closed. Every source is a bespoke repository
adapter (`git.repository.ts`, `claude-code.repository.ts`, `cursor.repository.ts`,
…) and `ActivitySource` is a hardcoded TypeScript union — adding a source means
editing Chronicle's code and cutting a release. The activity contract
(`Activity`, `Evidence`, `DailyChronicleData`) exists only as internal TS
interfaces; nothing outside the repo can validate against it.

This inverts the platform's own philosophy. Rosetta is AI-native — agents are
first-class producers and consumers of knowledge (ADR-0001) — yet an agent
today has no way to say "I did this, record it" without a human building an
adapter first. API-first means the contract is the product: publish the schema,
open the ingestion boundary, and Chronicle becomes a substrate any tool can
feed rather than a fixed set of integrations.

### 1.2 Goals

- Publish a formal, versioned schema (JSON Schema) for `Activity`, `Evidence`, and the daily sidecar — generated from the TS types so code and schema can never drift.
- Accept schema-valid activity from producers Chronicle has never heard of, with zero Chronicle code changes per new producer.
- Expose ingestion (and later query) as MCP tools, so any agent that speaks MCP can contribute activity natively — agentic-first assumes API-first.
- Validate and attribute at the boundary: every ingested activity carries producer provenance and passes schema validation before it enters the ledger.
- Preserve local-first: ingestion lands in the user's personal Chronicle repo (ADR-0002); no hosted service is required.

### 1.3 Non-Goals

- Not a hosted SaaS ingestion API — the boundary is local (filesystem inbox + local MCP server). Remote/multi-tenant ingestion is a future consideration.
- Not replacing the existing pull adapters — git, Claude Code, Cursor, calendar, and notes keep working exactly as today. Push complements pull.
- Not a real-time streaming pipeline — ingestion is append-and-synthesize-later, matching Chronicle's daily cadence.
- Not an auth/identity system — within the local trust boundary, provenance is declarative. Signed provenance arrives with remote ingestion.
- Not schema for org knowledge — this covers activity intake. Org-side contracts stay with PRD-0006/PRD-0009.

### 1.4 Acceptance Criteria

**Phase 1 — Formal schema & open source type:**

- [ ] JSON Schema documents for `Activity`, `Evidence`, and `DailyChronicleData` are generated from the TS types at build time and published to a shared location with a semver-style version (`activity.v1.schema.json`).
- [ ] `ActivitySource` accepts namespaced external identifiers (`ext:<namespace>/<tool>`, e.g. `ext:acme/deploy-bot`) alongside the known built-in sources, and render/synthesis layers handle them generically.
- [ ] A schema round-trip test proves every activity Chronicle itself produces validates against the published schema.

**Phase 2 — Inbox ingestion:**

- [ ] A producer can drop a schema-valid activity envelope (NDJSON) into a well-known inbox location in the personal Chronicle repo and the next synthesis run includes it, attributed to its producer.
- [ ] Invalid envelopes are rejected to a dead-letter location with a machine-readable validation error — never silently dropped, never partially ingested.
- [ ] Duplicate submissions (same source + id) are idempotent.

**Phase 3 — MCP server:**

- [ ] A `chronicle-mcp` server exposes `chronicle_ingest_activity` and `chronicle_list_schema` tools speaking the MCP `2026-07-28` stateless spec.
- [ ] An off-the-shelf MCP client (e.g. Claude Code, Cursor) can ingest an activity end-to-end with no Chronicle-specific code.
- [ ] Ingested-via-MCP activity is indistinguishable in the ledger from inbox-ingested activity (same envelope, same validation, same provenance fields).

## 2. Users & Motivation

**Primary:** agents. An autonomous agent that just deployed a service, triaged
an incident, or ran an experiment should record that activity itself — in the
moment, with evidence — rather than hoping a pull adapter reconstructs it later.

**Secondary:** engineers wiring up long-tail sources. A one-off shell script
piping CI results into the inbox beats waiting for a first-class adapter.

**Tertiary (and the payoff):** future devices and runtimes. A capture-only
device (see PRD-0016) participates in Chronicle by emitting schema-valid
activity — it needs the schema and the envelope, not an adapter, not a model.

Pain removed:

- **No more "Chronicle doesn't support X":** if it can emit JSON, it can contribute.
- **No adapter release cycle per source:** the schema is the integration surface.
- **No lost agent activity:** agents self-report with evidence at the moment of action.

## 3. Approach

### 3.1 Schema as the contract

The TS types in `rosetta_chronicle/src/types.ts` remain the source of truth.
A build step (e.g. `ts-json-schema-generator`) emits JSON Schema (draft
2020-12) artifacts, versioned and published where both humans and machines can
reach them (`rosetta_docs/shared/schemas/` initially; `rosetta_core` once
ADR-0004 lands). Schema versions are additive within a major version;
breaking changes bump the major (`activity.v2.schema.json`) and both versions
remain published.

### 3.2 Opening the source type

```
Today:  ActivitySource = 'git' | 'jira' | 'claude-code' | 'cursor' | 'notes' | 'calendar'
After:  ActivitySource = KnownActivitySource | ExternalActivitySource
        ExternalActivitySource = `ext:${namespace}/${tool}`
```

Built-in sources keep their first-class rendering (repo grouping for git,
session titles for agent transcripts). External sources render generically:
grouped by producer, summary + evidence, same needs-review pathway.

### 3.3 Ingestion pipeline (HSR)

```
Producer (script / agent / MCP client / device)
        │  ActivityEnvelope (NDJSON)
        ▼
┌──────────────────────────────┐
│ IngestionHandler             │  parse envelope, dispatch
│   └─ IngestionService        │  schema-validate, dedup (source+id),
│        │                     │  stamp receivedAt, enforce provenance
│        ├─ InboxRepository    │  read/ack inbox entries, dead-letter
│        └─ ChronicleStoreRepo │  append to the day's structured sidecar
└──────────────────────────────┘
        ▼
Daily synthesis unions inbox activity with pull-adapter activity
```

Transport is deliberately layered: the **inbox directory** (append-only NDJSON
files under the personal Chronicle repo, e.g. `inbox/YYYY-MM-DD/*.ndjson`) is
the transport-agnostic core — trivially writable by a shell script, a cron
job, or an offline device syncing later. The **MCP server** is a thin front
door that validates and writes to the same inbox, exposing the capability to
the agent ecosystem. Any future transport (HTTP endpoint, webhook relay)
fronts the same pipeline.

### 3.4 MCP surface

Targeting the MCP `2026-07-28` spec (stateless core, per-request version and
capabilities — a good match for a local, sessionless ingestion tool):

| Tool                        | Purpose                                                                                             |
| --------------------------- | --------------------------------------------------------------------------------------------------- |
| `chronicle_ingest_activity` | Submit one or more `ActivityEnvelope`s. Returns per-item accept/reject with validation errors.      |
| `chronicle_list_schema`     | Return the current published schema (id, version, JSON Schema body) so producers can self-validate. |
| `chronicle_query` _(later)_ | Read-side: query the sidecar by window/source/tag. Pairs with Wayfinder's needs.                    |

## 4. Data Contracts

```ts
/** Namespaced identifier for a producer Chronicle has no adapter for. */
export type ExternalActivitySource = `ext:${string}/${string}`;

/** Wire envelope for pushed activity. This — not bare Activity — is what
 *  producers submit, so provenance and versioning are mandatory. */
export interface ActivityEnvelope {
  /** Schema the payload claims conformance to, e.g. "activity.v1". */
  schemaVersion: string;
  /** Who produced this. */
  producer: {
    /** Matches the activity's source, e.g. "ext:acme/deploy-bot". */
    source: ExternalActivitySource;
    /** Producer software name/version for audit. */
    name: string;
    version?: string;
  };
  /** ISO-8601 time the producer emitted the envelope (may predate receipt —
   *  offline devices sync later). */
  emittedAt: string;
  /** The activity itself; must validate against the declared schema. */
  activity: Activity;
}

export interface IngestResult {
  /** source + id of the submitted activity. */
  ref: { source: string; id: string };
  outcome: "accepted" | "duplicate" | "rejected";
  /** Machine-readable validation errors when rejected. */
  errors?: string[];
}
```

## 5. Constraints & Dependencies

- **Architecture:** HSR + InversifyJS. Ingestion is Handler → Service → Repository; schema generation is a build concern, not runtime.
- **Privacy (ADR-0002):** ingestion targets the _personal_ Chronicle. Promotion to org knowledge still goes through PRD-0006/PRD-0009 — an open ingestion door does not bypass the coherence gate.
- **Schema/code coupling:** the schema must be generated from the TS types, never hand-maintained, or the two will drift.
- **MCP spec:** `2026-07-28` (stateless, header-carried version). Pin the SDK; the spec now has a formal deprecation policy so this is a stable target.
- **Clobber guard (PRD-0005):** inbox activity counts as collected activity for subset-detection — a regeneration must not drop previously ingested items.

## 6. Risks & Open Questions

- **Garbage in:** open ingestion means low-quality or spammy activity can pollute a chronicle. Mitigation: `reviewNeeded` pathway already exists; consider per-producer volume caps and a quarantine view before synthesis includes a first-seen producer.
- **Schema evolution pressure:** external producers freeze on published versions; Chronicle must support N and N-1 majors simultaneously. Keep the schema small and additive.
- **Tag inference for unknown sources:** the tag taxonomy inference is tuned for known source shapes. Generic external activity may tag poorly at first — acceptable, tags can be carried/corrected on regeneration.
- **Envelope trust:** within the local boundary, provenance is self-declared. Is that enough until remote ingestion? (Proposed: yes — same trust model as the existing filesystem sources.)
- **Where does the schema live long-term?** `rosetta_docs/shared/schemas/` vs `rosetta_core` (ADR-0004). Start in docs, move when core exists.

## 7. Rollout & Phases

1. **Phase 1 — Formal schema & open source type:** generate + publish JSON Schema from the TS types; widen `ActivitySource` with `ext:` namespacing; generic rendering for external sources; round-trip validation test.
2. **Phase 2 — Inbox ingestion:** `ActivityEnvelope`, inbox directory protocol, validation/dedup/dead-letter, synthesis unions inbox activity.
3. **Phase 3 — MCP server:** `chronicle-mcp` with ingest + schema tools on MCP `2026-07-28`; documented quick-start for wiring into Claude Code/Cursor.

## 8. Future Considerations

- **Read-side API:** `chronicle_query` MCP tool over the structured sidecar — the same open-boundary treatment for consumers that this PRD gives producers. Wayfinder and PRD-0012's tool belt are natural clients.
- **Remote ingestion:** an authenticated HTTP endpoint (or MCP-over-HTTP with OAuth per the 2026 spec) so producers off the local machine can contribute — requires signed provenance.
- **Producer registry:** a lightweight registry of known `ext:` namespaces with display names/icons for richer rendering.
- **Org-level ingestion:** teams pushing activity into an org chronicle directly (CI systems, incident bots) — gated by the coherence protocol (PRD-0009).
- **Capture-only devices:** the envelope + inbox protocol is the entire integration surface a wearable needs — see PRD-0016.
