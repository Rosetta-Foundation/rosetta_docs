---
id: PRD-0006
title: Artifact Capture & Promotion to Org Knowledge
status: Proposed
date: 2026-07-24
owner: Russ Watson
related_adrs: [ADR-0002]
related_specs: []
supersedes: null
---

# PRD-0006: Artifact Capture & Promotion to Org Knowledge

> Link the durable outputs of engineering work (diagrams, docs, designs) to the
> sessions that produced them, and provide a pipeline to promote private Chronicle
> knowledge into shared organizational memory.

## 1. Overview & Goals

### 1.1 Purpose

Chronicle today captures **activity** — commits, Claude sessions, meetings, notes.
It does not capture **artifacts** — the actual outputs of that work. An engineer
who spends an hour investigating a system and produces an architecture diagram
gets a one-line session summary ("Analyze Maximum labels calculation in Quota
status") but no link between that session and the diagram it produced. The diagram
is the valuable organizational knowledge; the session is just the provenance.

Worse, this knowledge is trapped in the private Chronicle. The architecture
diagram explains how the quota-limit resolution pipeline works across three tiers
of DynamoDB tables — knowledge any engineer touching that system needs. Today
there is no pathway from "I captured this in my private log" to "the org can
query this."

This capability closes both gaps: (1) link artifacts to the sessions/activity
that produced them, so the private Chronicle is a complete record of not just
*what you did* but *what you produced*, and (2) provide a promotion pipeline that
can surface selected private knowledge into a shared org layer — redacting
personal context and preserving the durable architectural insight.

### 1.2 Goals

- Link engineering artifacts (documents, diagrams, design files, architecture records) to the Chronicle activity that produced them.
- Make artifacts queryable in the private Chronicle: "what did I produce last week?" not just "what did I do?"
- Provide a promotion pathway: private → org knowledge, with explicit consent and content redaction.
- Preserve provenance: promoted knowledge links back to the evidence (session, commit, PR) that generated it.
- Keep the private/org boundary explicit and human-controlled — nothing is promoted without an intentional act.

### 1.3 Non-Goals

- Not auto-publishing anything — promotion is always an explicit human decision.
- Not a document management system — Chronicle links to artifacts where they live (in repos, in docs/), it doesn't ingest or store copies.
- Not replacing ADRs or PRDs — those are authoring surfaces; this is the knowledge *produced during* engineering work that doesn't rise to an ADR/PRD but is still valuable.
- Not building Wayfinder's query layer — this PRD defines the structured data Wayfinder would consume.
- Not cross-team aggregation (multi-user org chronicles) — that's a separate capability; this covers the single-user private→org promotion path.

### 1.4 Acceptance Criteria

**Phase 1 — Artifact linking in the private Chronicle:**

- [ ] An `Artifact` type exists in `types.ts` with fields: `path`, `type` (diagram, doc, design, etc.), `title`, `producedBy` (activity id), `repo`.
- [ ] The structured sidecar (`.data/<date>.json`) gains an `artifacts` array alongside `activities`.
- [ ] A CLI flag or notes-file convention allows declaring "this session produced this file" — e.g. `--artifact <path>` or a `[artifact: path/to/file]` line in the notes.
- [ ] The rendered Chronicle gains a "Produced" or "Artifacts" section when artifacts exist for the day.
- [ ] Re-running for the same day preserves previously-linked artifacts (no clobber).

**Phase 2 — Promotion pipeline:**

- [ ] A `/promote` skill or CLI command takes an artifact + its activity context and produces a redacted org-knowledge entry.
- [ ] Promoted entries land in a shared location (e.g. `rosetta_docs/knowledge/` or a Wayfinder-consumed store).
- [ ] Each promoted entry links back to its evidence (session id, commit SHA, PR URL) without exposing the private Chronicle.
- [ ] Redaction rules strip personal context (session titles mentioning people, private repo paths) while preserving architectural content.

## 2. Users & Motivation

**Anchor example:** On 2026-07-24, an engineer investigated why a usage dashboard
always showed hardcoded rate limits regardless of DynamoDB-stored per-user
quotas. The investigation produced:

- A Claude session: "Analyze Maximum labels calculation in Quota status"
- A fix: PR #72 (field name mismatch between consumer and reader)
- An architecture diagram: `example_usage_console/docs/architecture/quota-limit-resolution.html`

Chronicle captured the session and the fix (via commit + PR evidence). It did
**not** capture the diagram as an output of that session. The diagram documents
the three-tier resolution strategy (manager-approved → DynamoDB quotas →
hardcoded defaults) — exactly the kind of architectural knowledge that:

1. Would have prevented this bug if it existed before someone had to rediscover it
2. Any engineer touching the quota system needs access to
3. Is currently invisible to the organization unless someone manually shares it

**Serves:**
- The engineer (complete private record — "I produced this diagram during that investigation")
- The team (org-visible architectural knowledge, discoverable and queryable)
- Future agents (structured artifact→evidence links feed Wayfinder "how does X work?" queries)

## 3. Approach

Two tiers, building on the existing structured sidecar (PRD-0002):

**Tier 1 — Artifact linking (private).**

Artifacts are metadata — a pointer to a file + its provenance (which activity
produced it). They are stored in the sidecar alongside activities. Declaration is
lightweight: a notes-file convention (`[artifact: relative/path]`), a CLI flag
(`--artifact path`), or automatic detection (files created during a session that
match known patterns like `docs/**/*.md`, `*.html`, `*.svg` in architecture
directories).

The rendered Chronicle gains a section:

```markdown
## Artifacts Produced

- 📄 [quota-limit-resolution.html](../example_usage_console/docs/architecture/quota-limit-resolution.html) — architecture diagram (from session "Analyze Maximum labels...")
```

**Tier 2 — Promotion pipeline (private → org).**

A promotion step takes a private artifact + its context and produces an
org-knowledge entry:

```
Private Chronicle (input):
  Activity: "Analyze Maximum labels calculation"
  Artifact: docs/architecture/quota-limit-resolution.html
  Evidence: PR #72, session c6ee47ae
  Tags: [ARCH, RELIABILITY]

Org Knowledge (output):
  Title: "Quota Limit Resolution — Three-Tier Strategy"
  Summary: "Daily/weekly/monthly limits resolve via: (1) manager-approved
            DynamoDB request, (2) per-user DynamoDB quota row, (3) hardcoded
            60M/200M/600M defaults."
  Artifact: link to the diagram
  Evidence: PR #72
  Tags: [ARCH, RELIABILITY]
  Redacted: session id, personal chronicle path
```

The promotion is explicit (skill invocation or CLI command), human-reviewable
(shows what will be published before committing), and additive (never modifies
the private Chronicle).

## 4. Data Contracts

```ts
/** A durable output of engineering work, linked to the activity that produced it. */
export interface Artifact {
  /** Repo-relative path to the artifact file. */
  path: string;
  /** Classification: diagram, doc, design, config, test, etc. */
  type: ArtifactType;
  /** Human-readable title (derived from filename or explicit). */
  title: string;
  /** Activity id that produced this artifact (session id, commit SHA). */
  producedBy: string;
  /** Repository the artifact lives in. */
  repo?: string;
}

export type ArtifactType =
  | 'diagram'
  | 'doc'
  | 'design'
  | 'config'
  | 'test'
  | 'other';

/** Extended sidecar: activities + artifacts for a day. */
export interface DailyChronicleData {
  window: ChronicleWindow;
  tags: Tag[];
  activities: Activity[];
  artifacts: Artifact[];
}

/** An org-knowledge entry produced by promotion. */
export interface OrgKnowledgeEntry {
  /** Stable id for dedup. */
  id: string;
  title: string;
  summary: string;
  /** Link to the artifact (URL or repo-relative path). */
  artifactRef: string;
  /** Evidence from the private Chronicle, sanitized. */
  evidence: Evidence[];
  tags: Tag[];
  /** When the knowledge was produced (activity timestamp). */
  producedAt: string;
  /** When it was promoted to org. */
  promotedAt: string;
}
```

## 5. Constraints & Dependencies

- **Architecture:** Handler / Service / Repository + InversifyJS. Artifact linking
  is handler-level (compose with existing sidecar persistence). Promotion is a new
  service (redaction + rendering) with a new repository (org-knowledge store).
- **Privacy (ADR-0002):** private Chronicle stays private. Promotion is opt-in,
  human-reviewed, and redacts personal context. Nothing crosses the boundary
  without explicit consent.
- **Depends on:** PRD-0002 (structured sidecar — the substrate artifacts are stored
  in), PRD-0005 (clobber guard — artifact preservation across regeneration).
- **Relates to:** Wayfinder (the future query layer over promoted org knowledge).

## 6. Risks & Open Questions

- Artifact detection heuristic: explicit declaration only, or auto-detect files
  created during a session? Auto-detect risks false positives (build outputs,
  temp files); explicit requires engineer discipline.
- Promotion granularity: one artifact at a time, or batch ("promote everything
  tagged ARCH from this week")?
- Org knowledge format: plain Markdown in a repo (`rosetta_docs/knowledge/`), or a
  structured store Wayfinder consumes directly? Start with Markdown for human
  review; add structured indexing later.
- Redaction rules: what counts as "personal context"? Session titles, note text
  mentioning people, private repo paths. Need a clear policy.
- Multi-artifact sessions: one session may produce several related artifacts (a
  diagram + a doc + a PR). Link them as a group, or individually?
- Cross-repo artifacts: the diagram lives in `example_usage_console` but the
  Chronicle lives in `rosetta_chronicle_example-user`. Paths must be
  workspace-relative or absolute.

## 7. Rollout & Phases

1. **Phase 1 — Artifact linking in the private Chronicle:** extend the sidecar
   with an `artifacts` array; add a notes-file convention and/or CLI flag for
   declaration; render an "Artifacts Produced" section in the daily Markdown;
   preserve artifacts across regeneration (clobber guard extension). Closes the
   "what did I produce?" gap in the private Chronicle.
2. **Phase 2 — Promotion pipeline (private → org):** a `/promote` skill or CLI
   command that takes a private artifact + context, applies redaction rules,
   produces an org-knowledge entry in `rosetta_docs/knowledge/` (or a
   Wayfinder-consumed store), and commits it. Explicit, human-reviewed, additive.
3. **Phase 3 — Wayfinder integration (deferred):** structured indexing of promoted
   knowledge for "how does X work?" queries. This is a Wayfinder PRD, not a
   Chronicle one — Chronicle's job is done once the structured data exists.

## 8. Future Considerations

- Auto-detection of artifacts from session filesystem activity (files created or
  modified during a Claude Code session, filtered by pattern).
- Bi-directional links: from the artifact back to the Chronicle entry that
  produced it (e.g. a comment in the HTML file linking to the session).
- Versioned knowledge: when an architecture diagram is updated, the org entry
  should reflect the latest version while preserving history.
- Team-level Chronicle: aggregating promoted knowledge across multiple engineers'
  private Chronicles into a team/org-level view.
- AI-assisted promotion: an agent that suggests "this looks like reusable
  architectural knowledge — promote it?" based on tags and artifact type.
