---
id: PRD-0002
title: Multi-Repo Git Discovery & Two-Tier Daily Synthesis
status: Shipped
date: 2026-07-23
owner: Russ Watson
related_adrs: [ADR-0002]
related_specs: []
supersedes: null
---

# PRD-0002: Multi-Repo Git Discovery & Two-Tier Daily Synthesis

> Capture git activity across every repository under a workspace root, and build
> the Daily Chronicle by synthesizing durable per-repo structured data rather
> than re-parsing rendered Markdown.

## 1. Overview & Goals

### 1.1 Purpose

Chronicle's Claude Code source already discovers activity across a whole
workspace — it prefix-matches every session under a root directory. The Git
source does not: `GitRepository.getActivity(repoPath, window)` reads exactly one
repository, so "look at my laptop for all my work starting at the rootmost
project directory" is only true for Claude sessions, not commits. Covering
multiple repos today means multiple runs, and because the service rebuilds the
git section per run while only merging tags/notes back from rendered Markdown
(a lossy re-parse), each run **clobbers** the prior repo's commit section.

This capability closes that asymmetry and, in doing so, fixes the lossy merge:
git discovery becomes root-relative like Claude sessions, and the daily document
is synthesized from durable **structured** per-repo data instead of from
Markdown.

### 1.2 Goals

- Discover every git repository under a workspace root and include all their in-window commits in one Daily Chronicle, each attributed to its source repo.
- Persist durable, per-repo **structured** activity artifacts (not just rendered Markdown) as the source of truth.
- Synthesize the unified daily document from those structured artifacts, so re-runs never drop or clobber prior data.
- Keep the human-facing Markdown a pure render of structured truth.

### 1.3 Non-Goals

- Not aggregating commits across machines or remotes — local filesystem discovery under one root only (org-wide/GitHub-authored aggregation is a future GitHub source).
- Not changing the Claude Code or Notes sources' behavior beyond feeding the new synthesis tier.
- Not building a query/read UI over the structured artifacts (that is Wayfinder).
- Not organizational publication — output stays in the private Chronicle (ADR-0002).
- Not deep repo crawling without bounds — discovery is depth- and ignore-limited.

### 1.4 Acceptance Criteria

- [x] A `GitDiscoveryRepository` returns every git repository path under a given root, bounded by a max depth and a configurable ignore list (e.g. `node_modules`, `dist`, `.git` internals).
- [x] `ChronicleService` (or a discovery-aware coordinator) aggregates in-window commits from **all** discovered repos into one Daily Chronicle.
- [x] Each git `Activity` is attributed to its origin repository, and the rendered "Work Completed" section groups or labels commits by repo.
- [x] Per-repo structured artifacts are persisted (`Activity[]` + tags) under a stable, date-scoped path, one file per repo.
- [x] A `SynthesisService` builds the unified `DailyChronicle` by reading the per-repo structured artifacts for a date — not by parsing rendered Markdown.
- [x] Re-running backfill for a date is idempotent and non-destructive: no repo's commit section is clobbered, and existing tags/notes are preserved from structured data.
- [x] The lossy Markdown re-parse (`chronicle-parse.utils`) is no longer on the critical path for merging prior state.
- [x] Merge-commit inclusion is configurable, never hardcoded.

## 2. Users & Motivation

Serves the engineer first: "one command, from my project root, captures
everything I did today across all my repos" — no per-repo runs, no silent
clobbering. Serves future managers and agents by making per-repo activity a
first-class, structured, queryable artifact rather than prose locked in
Markdown. Removes two pains: the multi-repo blind spot in git, and the fragility
of reconstructing state from rendered output.

## 3. Approach

Two tiers, both within Handler / Service / Repository + InversifyJS.

**Tier 1 — Discovery & aggregation (Phase 1).**

- `GitDiscoveryRepository` — resource access only: walk the filesystem under a
  root, return repo paths (depth-limited, ignore-list aware).
- `GitRepository` stays single-repo (resource access to one repo's `git log`).
- The service orchestrates: discover repos → `getActivity` per repo → aggregate,
  attributing each `Activity` to its repo. This is legitimate service
  composition of repositories; no service calls another service.

**Tier 2 — Structured sidecars & synthesis (Phase 2).**

- A `ChronicleStore` repository reads/writes **structured** per-repo artifacts
  (JSON) — the durable source of truth.
- A `SynthesisService` aggregates the per-repo structured artifacts for a date
  (plus workspace-wide Claude sessions and notes) into the unified
  `DailyChronicle`, then reuses `render.utils` to produce Markdown.
- The handler composes "generate per-repo structured data → persist → synthesize
  → render → persist Markdown."

Proposed on-disk layout (per-repo storage feeding a unified daily):

```
chronicles/
├── 2026-07-21.md                       unified human render
└── .data/
    └── 2026-07-21/
        ├── rosetta_chronicle.json       per-repo structured activity + tags
        ├── rosetta_dev-scripts.json
        ├── rosetta_wayfinder.json
        └── _synthesis.json              aggregated activity + unioned tags
```

## 4. Data Contracts

```ts
// Resource access: discover git repos under a root.
export interface IGitDiscoveryRepository {
  discover(root: string, opts?: DiscoveryOptions): Promise<string[]>;
}

export interface DiscoveryOptions {
  maxDepth?: number; // bound the walk
  ignore?: string[]; // directory names to skip
  includeMerges?: boolean; // never hardcoded; passed through to git log
}

// Attribute activity to its origin repository.
export interface Activity {
  // ...existing fields...
  /** Slug of the repository this activity originated from (git source). */
  repo?: string;
}

// Durable structured artifact persisted per repo, per day.
export interface RepoChronicleData {
  window: ChronicleWindow;
  repo: string;
  activities: Activity[];
  tags: Tag[];
}

// Structured store — the source of truth the synthesis reads from.
export interface IChronicleStore {
  writeRepoData(repoPath: string, data: RepoChronicleData): Promise<void>;
  readDay(repoPath: string, date: string): Promise<RepoChronicleData[]>;
}
```

Reuses existing `Activity` / `Evidence` / `Tag` / `ChronicleWindow` /
`DailyChronicle` types (`src/types.ts`).

## 5. Constraints & Dependencies

- **Architecture:** Handler / Service / Repository + InversifyJS. Discovery and
  structured I/O are repositories (resource access only); aggregation and
  synthesis are services; the handler composes. No service-calls-service.
- **Privacy (ADR-0002):** local, project-scoped reads; private-repo destination.
- **Team standard:** merge-commit filtering is configurable, never hardcoded
  `--no-merges`.
- **Backward compatibility:** existing `chronicles/<date>.md` output path is
  preserved; the `.data/` sidecar is additive.
- **Depends on:** the existing single-repo `GitRepository` and the source
  pattern established by PRD-0001 (Claude Code source).

## 6. Risks & Open Questions

- Repo attribution key: directory basename vs. git remote/origin slug — basename
  is simplest but collides across nested checkouts.
- Discovery cost on large trees — needs sane default `maxDepth` and ignore list;
  should discovery cache within a backfill run?
- Migration: existing Markdown-only chronicles have no `.data/` sidecar — does
  synthesis backfill sidecars from Markdown once, or treat pre-feature days as
  Markdown-only?
- Structured artifact format/versioning — JSON schema evolution as `Activity`
  gains fields.
- Nested repos / submodules — discover both, or stop at the outermost `.git`?
- Should the `.data/` sidecars be committed to the Chronicle repo, or gitignored
  as a local cache and regenerable from sources?

## 7. Rollout & Phases

1. ✅ **Phase 1 — Multi-repo git discovery (flat unified daily):**
   `GitDiscoveryRepository` walks the root; the service aggregates all repos'
   in-window commits into one Daily Chronicle with per-repo attribution. Closes
   the git/Claude asymmetry. No new storage tier — renders straight to Markdown
   as today. Answers "look at my laptop from the root."
2. ✅ **Phase 2 — Two-tier structured sidecars & synthesis:** add `ChronicleStore`
   (structured per-repo JSON) and `SynthesisService`; persist per-repo data,
   synthesize the unified daily from structured truth, retire the lossy Markdown
   re-parse for state merges.

## 8. Future Considerations

- GitHub source: org-wide, authored-by-me aggregation across machines/remotes
  (the network counterpart to local discovery).
- Feeding the structured `.data/` artifacts directly into Wayfinder queries
  ("what did I do in repo X last week?").
- Live append (Stop-hook) writing structured sidecars incrementally as work
  happens, not only in batch — aligns with the live-sourcing direction.
- Model-assisted synthesis (narrative rollup across repos) once structured data
  is the source of truth.
