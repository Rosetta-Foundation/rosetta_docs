---
id: PRD-0019
title: Shared Chronicle Provisioning & Multi-Destination Promotion
status: Draft
date: 2026-08-02
owner: Russ Watson
related_adrs: [ADR-0002, ADR-0005]
related_specs: []
supersedes: null
---

# PRD-0019: Shared Chronicle Provisioning & Multi-Destination Promotion

> Let people create and join shared chronicles, choose where personal knowledge
> is promoted, and review machine suggestions — without ever auto-publishing.

## 1. Overview & Goals

### 1.1 Purpose

ADR-0002 settles the seam: knowledge is born personal; drafting may be
automatic; publishing is intentional. ADR-0005 settles the scale shape: team,
org, community, and global chronicles are the same engine over different
repositories and policies. PRD-0006 defines artifact linking and a single
promote primitive into “the org.” PRD-0009 defines the coherence gate that runs
when a promotion lands in a shared repository.

What remains unspecified — and blocks the essay’s “personal → common record”
story from becoming machinery — is the **lifecycle of shared chronicles** and
the **destination model** for promotion:

- Where do org/community chronicles come from?
- How does a person choose one or more destinations?
- How does the machine surface promotion candidates — and how does a person
  find what the machine missed?
- Can one intentional promote land in multiple shared chronicles?

This PRD owns those questions. It does **not** replace redaction rules
(PRD-0006) or per-destination coherence (PRD-0009).

### 1.2 Goals

- Provide a personal **destination registry**: the shared chronicles a person
  may promote into (joined or authorized), with stable ids and remotes.
- Support **multi-destination promotion**: one intentional human act may target
  a non-empty set of destinations; each destination runs its own coherence gate.
- Define **create / join / leave** lifecycle for team, org, and community
  chronicles using git semantics (ADR-0005) — no privileged central directory
  required for core function.
- Define a **suggestion inbox** for machine-proposed promotion candidates
  (ADR-0002 “drafting is automatic”) that never publishes without human action.
- Define a **missed-review surface** so a person can browse their personal
  chronicle for promote-eligible material the suggester did not raise.
- Keep first-class personal ledger states for promotion hygiene: suggested,
  dismissed, snoozed, promoted (per destination), never-promote.

### 1.3 Non-Goals

- Not changing the human gate — nothing becomes shared knowledge without an
  intentional approve (ADR-0002).
- Not defining redaction / sanitize rules for a single promote payload —
  that remains PRD-0006.
- Not defining contradiction / staleness / dedup detection — that remains
  PRD-0009, clarified as **per destination repository**.
- Not a hosted global search/directory that every node depends on
  (ADR-0005 exit test). Optional discovery conveniences are allowed later;
  core create/join/promote must work with remotes the user already knows.
- Not auto-fan-out to destinations the promoter did not select.
- Not replacing Wayfinder’s general query UX — this PRD defines promotion-
  specific surfaces Chronicle and/or Wayfinder may host.
- Not community-scale identity/reputation systems (deferred; ADR-0005 notes
  the hardness). Phase 1 trusts git authz + destination membership.

### 1.4 Acceptance Criteria

**Phase 1 — Destination registry & promote picker:**

- [ ] A personal-side `PromotionDestination` registry exists (config or ledger
      metadata) listing shared chronicles the user can promote into: `id`,
      `displayName`, `remote`, `tier` (`team` | `org` | `community` | `global`),
      `default?`, `joinedAt`.
- [ ] `/promote` (or equivalent CLI/skill from PRD-0006) requires an explicit
      destination set (or confirmed default); never silently picks “the org.”
- [ ] Promoting to N destinations produces N independent landings (commit or
      coherence PR per destination); partial success is allowed and reported.
- [ ] Each landing carries a shared `promotionId` for audit plus
      per-destination outcome (`landed` | `conflicted` | `rejected` | `failed`).
- [ ] Documentation states that PRD-0009 coherence runs **per destination**.

**Phase 2 — Suggestion inbox & missed-review:**

- [ ] A suggestion producer can write `PromotionSuggestion` records into the
      personal chronicle (never into a shared chronicle).
- [ ] An inbox surface lists open suggestions with: why suggested (signals),
      proposed redacted preview, proposed destination set (editable), dismiss /
      snooze / promote actions.
- [ ] A missed-review surface lists personal activities/artifacts that are
      promote-eligible and not yet in a terminal promotion state (`promoted` to
      at least one dest, `never-promote`, or actively `snoozed`).
- [ ] Dismiss and never-promote persist and suppress equivalent re-suggestions
      (same activity/artifact id; similar-content suppression may be heuristic).
- [ ] No suggestion path can publish without the same human approve step as
      manual promote.

**Phase 3 — Create / join shared chronicles:**

- [ ] A `chronicle create-shared` (name TBD) flow provisions a new shared
      ledger repo: empty/minimal ledger layout, chosen visibility policy,
      initial admin = creator, remote published to user-chosen host.
- [ ] A join flow adds an existing remote to the personal destination registry
      after authz succeeds (clone or add remote + membership check).
- [ ] Leave removes the destination from the registry; it does not delete the
      remote repository.
- [ ] Create/join works offline for local-only remotes / paths; hosted remotes
      are a convenience (ADR-0005).
- [ ] README / foundations glossary cross-link: promotion destinations are
      shared chronicles; provisioning is this PRD, not ADR-0002’s personal
      user-derived setup.

## 2. Users & Motivation

**Primary:** an engineer (or civic participant) whose personal chronicle holds
reusable knowledge and who belongs to one or more shared communities of
practice — employer org, open-source project, research circle, local civic
group.

**Pain removed:**

- “I have one vague ‘org’ bucket” → choose where knowledge belongs.
- “The machine never asked, so I never promoted” → suggestion inbox.
- “I don’t trust the machine’s taste” → missed-review browse.
- “Standing up a community memory requires a central product” → git-shaped
  create/join.

**Secondary:** maintainers of a shared chronicle who need promotions to arrive
as coherent, attributable contributions (PRD-0009), not as silent writes from
unknown pipelines.

## 3. Approach

### 3.1 Destination model

```text
Personal chronicle
        │
        │  destination registry (local)
        │
        ├── team chronicle A
        ├── org chronicle B
        └── community chronicle C
```

- Destinations are **references** (id + remote + policy metadata), not copies
  of the shared ledger inside the personal repo.
- Default destination is optional and user-set; first promote with a default
  still requires confirmation in Phase 1.
- Authorization: can-promote means the user can open the destination’s write
  path (direct push where allowed, otherwise fork/PR). Exact membership model
  is destination policy.

### 3.2 Multi-destination promote

```text
Human approves promote(payload, destinations[])
        │
        ├─► dest A → redact (PRD-0006) → coherence (PRD-0009) → land/PR
        ├─► dest B → redact (PRD-0006) → coherence (PRD-0009) → land/PR
        └─► dest C → …
```

Settled defaults for this PRD:

| Question                          | Decision                                          |
| --------------------------------- | ------------------------------------------------- |
| Multi-dest allowed?               | **Yes**, only for destinations the human selected |
| Fan-out without selection?        | **No**                                            |
| Shared `promotionId`?             | **Yes**, one id; per-dest outcomes recorded       |
| Dest A accepts, dest B conflicts? | **OK** — independence is required by ADR-0005     |
| One act vs N acts?                | **One intentional act**, N landings               |

### 3.3 Suggestion producer (automatic drafting)

Signals (Phase 2 starting set — tune for precision over recall):

- Artifact type in `{diagram, design, doc}` linked to activity (PRD-0006)
- Tags such as `ARCH`, `RELIABILITY`, `DECISION` (when present)
- Presence of org-shareable evidence (PR, commit on a non-personal remote)
- Exclusion: career/promotion-packet language, private reflection heuristics,
  content classes denied by healthcare/copyright guardrails (see PRD-0015/0017)

Output is always a personal `PromotionSuggestion`. Publishing still requires
human approve → PRD-0006 promote path → destinations[] → PRD-0009 per dest.

### 3.4 Missed-review surface

Complement to the inbox: query personal sidecars for items that look
promote-eligible (same signals, lower threshold or human filters: date range,
tag, repo, artifact type) and are not terminal. This is how a person catches
what the suggester missed without waiting for a notification.

### 3.5 Shared chronicle lifecycle

| Action       | Behavior                                                                                                                                           |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Create**   | Provision new git ledger repo + minimal layout + policy stub (who may promote, coherence strictness, visibility). Creator is initial admin.        |
| **Join**     | User adds remote; membership/authz verified; registry entry created.                                                                               |
| **Leave**    | Registry entry removed; remote repo untouched.                                                                                                     |
| **Discover** | Out of Phase 1–3 core: users paste remotes / org catalogs they already trust. Optional accelerator directories later must pass ADR-0005 exit test. |

Tiers (`team` | `org` | `community` | `global`) differ by **policy defaults**,
not machinery (ADR-0005).

### 3.6 Personal promotion states

Per activity or artifact id (and optionally per destination after promote):

| State           | Meaning                                                                       |
| --------------- | ----------------------------------------------------------------------------- |
| `none`          | Not yet considered                                                            |
| `suggested`     | Open suggestion in inbox                                                      |
| `snoozed`       | Hidden until `snoozeUntil`                                                    |
| `dismissed`     | Rejected this suggestion instance; may reappear if content changes materially |
| `never-promote` | Terminal: do not suggest; exclude from missed-review default                  |
| `promoted`      | Landed or conflict-PR opened for a destination (record dest + outcome)        |

## 4. Data Contracts

```ts
export type ChronicleTier =
  "personal" | "team" | "org" | "community" | "global";

export interface PromotionDestination {
  /** Stable id in the personal registry (not necessarily the remote name). */
  id: string;
  displayName: string;
  /** Git remote URL or local path. */
  remote: string;
  tier: Exclude<ChronicleTier, "personal">;
  /** User-chosen default for confirm-to-promote flows. */
  isDefault?: boolean;
  joinedAt: string;
}

export type PromotionSuggestionSignal =
  "artifact-type" | "tag" | "org-evidence" | "heuristic";

export interface PromotionSuggestion {
  id: string;
  createdAt: string;
  /** Activity and/or artifact ids in the personal chronicle. */
  sourceActivityIds: string[];
  sourceArtifactIds?: string[];
  signals: PromotionSuggestionSignal[];
  rationale: string;
  /** Proposed destinations; human may edit before approve. */
  proposedDestinationIds: string[];
  /** Redacted preview fields — never write these to a shared repo here. */
  previewTitle: string;
  previewSummary: string;
  status: "open" | "snoozed" | "dismissed" | "promoted" | "expired";
  snoozeUntil?: string;
}

export type PromotionDestOutcome =
  "landed" | "conflicted" | "rejected" | "failed";

export interface PromotionRecord {
  /** Correlates landings across destinations for one human approve. */
  promotionId: string;
  approvedAt: string;
  approvedBy: string;
  sourceActivityIds: string[];
  sourceArtifactIds?: string[];
  destinations: Array<{
    destinationId: string;
    outcome: PromotionDestOutcome;
    /** Commit SHA, PR URL, or error detail. */
    ref?: string;
    coherenceCheckId?: string;
  }>;
}
```

## 5. Constraints & Dependencies

- **Architecture:** Handler / Service / Repository + InversifyJS. Registry and
  suggestion persistence are repositories; promote orchestration is a service
  that composes PRD-0006 redaction + per-dest PRD-0009 checks.
- **Privacy (ADR-0002):** suggestions and missed-review live only in the
  personal chronicle. Shared chronicles never pull personal content unprompted.
- **Decentralization (ADR-0005):** create/join/promote must satisfy the three
  design tests (no server required for core function; N=2 peers; same code path
  as personal scale with different policy).
- **Depends on:** PRD-0006 (promote payload + redaction), PRD-0009 (coherence
  per dest), ADR-0002 (publication properties), ADR-0005 (tier sameness).
- **Relates to:** PRD-0010 / PRD-0012 (Wayfinder may host inbox UX), PRD-0007
  (queue patterns may inspire suggestion storage), PRD-0015/0017 (ineligible
  content classes).

## 6. Risks & Open Questions

- **Suggestion precision:** noisy suggestions train people to ignore the inbox.
  Start conservative; measure dismiss rate before expanding signals.
- **Authz diversity:** GitHub org membership, bare git remotes, and local paths
  differ. Phase 3 should define a minimal membership check interface with
  pluggable adapters rather than assuming GitHub.
- **Partial multi-dest success:** UX must make “2 landed, 1 conflicted” obvious
  so people do not assume global success.
- **Policy stubs at create time:** too many knobs block creation; too few force
  unsafe defaults. Ship a small policy template per tier.
- **Community identity:** spoofed promoters at community scale remain an
  open ADR-0005 concern — out of Phase 1–3 except “signed commits / host authz.”
- **Cross-destination coherence:** detecting that dest A and dest B contradict
  each other is federation (PRD-0009 §8), not this PRD.

## 7. Rollout & Phases

1. **Phase 1 — Destination registry & promote picker:** registry + explicit
   destinations on promote + multi-dest landings with `promotionId` + docs that
   PRD-0009 is per destination.
2. **Phase 2 — Suggestion inbox & missed-review:** personal suggestions,
   dismiss/snooze/never-promote, browse-what-was-missed, still human-gated
   publish.
3. **Phase 3 — Create / join shared chronicles:** provision shared ledger
   repos, join/leave registry flows, tier policy templates, ADR-0005-safe
   operation without a central directory.

## 8. Future Considerations

- Optional discovery accelerators (org catalog, community index) that remain
  rebuildable / dismissable under ADR-0005.
- Cross-destination federation / coherence (extends PRD-0009 §8).
- Contributor trust levels that affect auto-land rates inside a destination
  (PRD-0009 Phase 3) — still per destination.
- Wayfinder-native inbox with voice triage (composes with PRD-0017/0018).
- Promotion packets for career use stay personal forever; never suggested for
  shared destinations.
