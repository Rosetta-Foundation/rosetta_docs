---
id: PRD-0009
title: Org Knowledge Coherence Protocol
status: Proposed
date: 2026-07-24
owner: Russ Watson
related_adrs: [ADR-0002]
related_specs: []
supersedes: null
---

# PRD-0009: Org Knowledge Coherence Protocol

> A promotion-time gate that ensures org knowledge stays consistent, current, and
> conflict-free — without silently overwriting anyone's contributions.

## 1. Overview & Goals

### 1.1 Purpose

Rosetta's architecture splits knowledge into private (personal Chronicle) and
shared (org repository). PRD-0006 introduces promotion — moving artifacts from
private to org. But promotion at scale creates a governance problem:

- **Contradictions:** Alice promotes a diagram showing auth via Okta; Bob promotes
  one showing Entra. Which is current?
- **Staleness:** A promoted architecture doc references a service renamed last
  sprint. Nothing updates it.
- **Duplication:** Two engineers promote nearly-identical notes from the same
  meeting. The org repo accumulates redundant, slightly-divergent copies.
- **Overwrite risk:** A newer promotion silently clobbers an older one that
  contained information the new one doesn't.

Without a coherence layer, the org repo degrades into a pile of contradictory
snapshots — the opposite of durable knowledge. The coherence protocol makes
promotion safe by detecting conflicts at write-time and surfacing them for
resolution, while allowing clean contributions to land automatically.

### 1.2 Goals

- Detect factual contradictions, staleness, and duplication at promotion time — before content lands in the org repo.
- Auto-land clean promotions (no conflicts, no contradictions) without human intervention.
- Surface conflicts as annotated PRs with clear context so resolution is fast, not archaeological.
- Never silently overwrite — the protocol is append/propose, not clobber.
- Attribute all contributions to their source (person, session, date) so provenance is traceable.
- Keep the org repo converging toward a single source of truth per topic, not accumulating contradictory fragments.

### 1.3 Non-Goals

- Not a real-time sync system — coherence runs on promotion events, not continuously.
- Not enforcing a single writing style or format — contradictions are semantic (facts), not cosmetic.
- Not resolving conflicts autonomously (Phase 1) — the agent proposes, humans decide. Autonomous resolution is Phase 3.
- Not handling cross-org knowledge (multiple companies, external sources) — scoped to a single org repo.
- Not a permissions/access system — who *can* promote is separate from whether the content is coherent.

### 1.4 Acceptance Criteria

**Phase 1 — Conflict detection & gated promotion:**

- [ ] A coherence check runs on every promotion to the org repo (invoked by the `/promote` skill or `queue promote`).
- [ ] The check detects: factual contradictions (conflicting claims about the same entity), staleness (references to renamed/removed code artifacts), and duplication (semantically equivalent content already in the org repo).
- [ ] Clean promotions (no conflicts detected) auto-land: content is committed directly to the org repo with attribution metadata.
- [ ] Conflicted promotions open a PR with inline annotations explaining each conflict and its context (source, date, contributor).
- [ ] The protocol never overwrites existing org content without going through the conflict resolution path.

**Phase 2 — Staleness detection & refresh cycle:**

- [ ] A periodic (configurable) sweep detects stale org knowledge: references to symbols/files/services that no longer exist in the codebase.
- [ ] Stale entries are flagged with a `[stale:reason]` annotation and optionally opened as refresh PRs.
- [ ] Staleness detection cross-references the org repo against the current state of all tracked codebases (git repos in `shared.json`).
- [ ] Contributors to the original content are notified (via PR assignment or comment) to confirm or update.

**Phase 3 — Autonomous resolution & deduplication:**

- [ ] The coherence agent can autonomously resolve low-risk conflicts: deduplication (merge two equivalent entries), timestamp-based supersession (newer fact replaces older when both reference the same entity).
- [ ] High-risk conflicts (semantic disagreements, removal of information) still require human approval.
- [ ] A confidence threshold governs autonomous resolution — configurable per org.
- [ ] Resolved conflicts produce an audit trail: what was merged, what was superseded, why, and by whom (or which agent rule).

## 2. Users & Motivation

**Primary:** any engineer promoting knowledge from their personal Chronicle to the org repo. They need confidence that their contribution won't silently break existing knowledge, and that conflicts will be surfaced quickly rather than festering.

**Secondary:** the team consuming org knowledge. They need to trust that what's in the org repo is current, non-contradictory, and attributed — not a graveyard of outdated fragments.

**Future:** agents querying org knowledge (Wayfinder). They need a single coherent answer per topic, not multiple conflicting documents to arbitrate between at query time.

Pain removed:
- **No more "which doc is current?"** — the protocol ensures only one coherent version exists per topic.
- **No more silent overwrite** — every conflict is visible and attributed.
- **No fear of promoting** — the gate catches problems, so engineers promote freely rather than self-censoring.

## 3. Approach

### 3.1 The Promotion Gate

Every promotion goes through a coherence check before landing:

```
Engineer promotes artifact
         │
         ▼
┌─────────────────────┐
│   Coherence Agent   │
│                     │
│  1. Semantic diff   │──── Compare incoming content against existing
│  2. Entity resolve  │──── Match claims to known entities (services, APIs, ADRs)
│  3. Freshness check │──── Cross-ref code artifacts (do they still exist?)
│  4. Dedup scan      │──── Semantic similarity against existing org docs
│                     │
└─────────┬───────────┘
          │
    ┌─────┴─────┐
    │           │
  Clean      Conflict
    │           │
    ▼           ▼
Auto-commit   Open PR with
to org repo   annotations
```

### 3.2 Conflict Types & Detection

| Type | Detection method | Resolution path |
|------|-----------------|-----------------|
| **Factual contradiction** | Claim extraction + entity matching: "Service X uses auth method Y" vs existing "Service X uses auth method Z" | PR with both claims, timestamps, sources. Human picks current truth. |
| **Staleness** | Symbol/path/service name resolution against tracked codebases. If a referenced identifier doesn't exist → stale. | Annotate with `[stale:symbol-not-found]`. Refresh PR assigned to original author. |
| **Duplication** | Semantic embedding similarity above threshold (e.g. >0.85 cosine) against existing org docs. | Merge: keep the richer version, attribute both contributors, archive the duplicate. |
| **Subset overwrite** | Incoming content covers fewer topics than the existing doc on the same subject (similar to PRD-0005 clobber guard). | Block auto-land. PR shows what would be lost. |

### 3.3 Attribution Metadata

Every org knowledge entry carries:

```ts
interface OrgKnowledgeAttribution {
  /** Who promoted this content. */
  promotedBy: string;
  /** When the promotion landed. */
  promotedAt: string;
  /** Source session/chronicle date. */
  sourceDate: string;
  /** Source type: personal chronicle, meeting notes, artifact, etc. */
  sourceType: 'chronicle' | 'artifact' | 'notes' | 'manual';
  /** SHA of the coherence check that approved landing. */
  coherenceCheckId?: string;
  /** If this superseded a prior version, link to it. */
  supersedes?: string;
}
```

### 3.4 Resolution Protocol

When a conflict is detected:

1. **Open a PR** in the org repo with:
   - The incoming content as the proposed change
   - Inline annotations on each conflict point
   - A summary comment: what conflicts, why, who contributed each side, timestamps
2. **Assign** the PR to:
   - The promoter (they authored the new content)
   - The original contributor (they authored the existing content) — if identifiable
3. **Resolution options:**
   - Accept incoming (supersede existing)
   - Keep existing (reject promotion)
   - Merge (combine both — agent can suggest a merged version)
   - Defer (close PR, re-open when more context available)
4. **On resolution:** update attribution, archive the superseded version (never delete), record the decision in an audit log.

### 3.5 Trust Levels (Phase 3)

| Level | Auto-land scope | Example |
|-------|----------------|---------|
| **High** | Factual updates with clear timestamp supersession | "Service X moved from Okta to Entra" with a date after the existing claim |
| **Medium** | Deduplication where one version is strictly richer | Two meeting notes; one has more detail but no contradictions |
| **Low** | Anything involving removal or semantic disagreement | Always requires human |

## 4. Data Contracts

```ts
export type ConflictType =
  | 'contradiction'
  | 'staleness'
  | 'duplication'
  | 'subset-overwrite';

export interface CoherenceConflict {
  type: ConflictType;
  /** What the incoming content claims. */
  incoming: string;
  /** What the existing org content claims (if applicable). */
  existing?: string;
  /** The entity or topic both reference. */
  entity: string;
  /** Confidence that this is a real conflict (0–1). */
  confidence: number;
  /** Suggested resolution (for agent-assisted merge). */
  suggestion?: string;
}

export interface CoherenceCheckResult {
  /** Unique id for audit trail. */
  checkId: string;
  /** Timestamp of the check. */
  checkedAt: string;
  /** Clean = auto-land; conflicted = open PR. */
  outcome: 'clean' | 'conflicted';
  /** Empty if clean. */
  conflicts: CoherenceConflict[];
  /** Attribution for the incoming content. */
  attribution: OrgKnowledgeAttribution;
}

export interface OrgKnowledgeAttribution {
  promotedBy: string;
  promotedAt: string;
  sourceDate: string;
  sourceType: 'chronicle' | 'artifact' | 'notes' | 'manual';
  coherenceCheckId?: string;
  supersedes?: string;
}
```

## 5. Constraints & Dependencies

- **Architecture:** HSR + InversifyJS. Coherence check is a Service; entity resolution and similarity are Repositories (resource access to embeddings, codebase, org repo).
- **Depends on:** PRD-0006 (artifact promotion — the trigger), PRD-0002 (structured sidecar — the machine-readable activity data to compare against), git (the org repo is a git repo; PRs are the conflict resolution UX).
- **Privacy (ADR-0002):** the coherence agent reads org content freely but only sees personal content when it's explicitly promoted. It never pulls from private chronicles unprompted.
- **LLM dependency:** claim extraction and semantic similarity require an LLM call. Must handle rate limits, cost, and latency gracefully (cache embeddings, batch checks).
- **Git-native:** conflicts are PRs. No custom UI needed — GitHub/GitLab PR review is the resolution interface.

## 6. Risks & Open Questions

- **Claim extraction quality:** LLM-based claim extraction may hallucinate conflicts that aren't real (false positives) or miss real ones (false negatives). Start conservative (flag more, auto-land less) and tune the confidence threshold over time.
- **Embedding cost at scale:** every promotion needs a semantic similarity check against all existing org docs. Mitigation: pre-compute and cache org doc embeddings; only recompute on change.
- **Resolution fatigue:** if too many promotions generate conflict PRs, engineers will ignore them. Mitigation: high auto-land rate for clean content; only genuine conflicts produce PRs.
- **Circular supersession:** A supersedes B, then later B supersedes A (flip-flopping). Mitigation: audit trail + "last N supersessions for this entity" visibility.
- **Multi-contributor merge:** more than two people have conflicting claims about the same entity. PR review with 3+ assignees gets complex. Start with pairwise (incoming vs. existing); generalize later.
- **Freshness signal for external systems:** staleness detection can cross-ref code repos, but can't easily check if a Jira ticket is still open or a Slack channel still exists. Scope to codebase-verifiable references initially.

## 7. Rollout & Phases

1. **Phase 1 — Conflict detection & gated promotion:** coherence check runs on every promotion. Detects contradictions, staleness, duplication. Clean → auto-land. Conflicted → annotated PR. No autonomous resolution.
2. **Phase 2 — Staleness detection & refresh cycle:** periodic sweep of org repo against tracked codebases. Stale entries flagged and assigned for refresh. Contributor notification.
3. **Phase 3 — Autonomous resolution & deduplication:** agent resolves low-risk conflicts (dedup, timestamp supersession) automatically. Configurable confidence threshold. Audit trail for all autonomous actions.

## 8. Future Considerations

- **Wayfinder integration:** when Wayfinder queries org knowledge and finds flagged conflicts, it can surface both sides with context rather than picking one arbitrarily.
- **Cross-org coherence:** when multiple teams have separate org repos, a federation layer that detects cross-boundary contradictions (e.g. shared service documentation diverging between consumer and provider).
- **Decay scoring:** org knowledge entries accumulate a "freshness score" that decays over time. Heavily-decayed entries are proactively flagged for review or archival.
- **Contributor reputation:** engineers who consistently promote clean, non-conflicting content earn a higher trust level → more auto-landing. Not gamification — just reducing friction for reliable contributors.
- **Real-time coherence for Wayfinder queries:** at query time, if the answer draws on multiple sources, run a lightweight coherence check and annotate confidence.
