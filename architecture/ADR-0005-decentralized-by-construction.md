# ADR-0005: Decentralized by Construction — Git Semantics at Every Scale

**Status:** Proposed

**Date:** 2026-07-31

---

> Every chronicle — personal, team, organizational, community, global — is the
> same machinery run against a distributed repository. No tier of the system
> may require a central server to function. Scale is achieved by federation
> (remotes, clones, merges, gates), never by centralization.

---

# Background

Rosetta is already decentralized in practice, by accident of its substrate:

- The ledger is git ([ADR-0002](ADR-0002-personal-vs-organizational-chronicle.md),
  [PRD-0010](../product/PRD-0010-wayfinder-local-app.md)) — every participant
  holds a full local replica, works offline, and syncs via remotes.
- Personal and organizational chronicles are **separate data repositories over
  one neutral engine** (ADR-0002) — the engine has no home server.
- The shared-core split ([ADR-0004](ADR-0004-shared-rosetta-core.md)) keeps the
  engine "machinery only" — contracts and pure logic with no privileged
  deployment.

But the property is nowhere stated as binding. Decentralization appears only in
scattered future considerations: federated org repos
([PRD-0010](../product/PRD-0010-wayfinder-local-app.md) §8), cross-org
coherence ([PRD-0009](../product/PRD-0009-coherence-protocol.md) §8), peer
synthesis ([PRD-0016](../product/PRD-0016-offline-on-device-intelligence.md) §8).
Nothing prevents a future feature — a search index, a discovery service, a
"global chronicle" — from being designed as a central service that every node
depends on. One such dependency quietly converts the platform from
git-shaped to hub-shaped, and the conversion is very hard to reverse.

The end game makes this urgent rather than theoretical: shared knowledge at
community or global scale **will** exist in some shape. The question this ADR
settles is whether that scale is reached with the same machinery (a very large,
heavily-gated chronicle repository that anyone can clone) or with different
machinery (a hosted platform that personal chronicles report to).

# Decision

## 1. One machinery, N repositories, zero privileged nodes

A chronicle at any scale is: **a data repository + the engine + policy.** The
tiers differ only in policy — who may write, what coherence gates apply
([PRD-0009](../product/PRD-0009-coherence-protocol.md)), what gets redacted on
promotion ([PRD-0006](../product/PRD-0006-artifact-capture-and-promotion.md)) —
never in machinery. The community/global chronicle, whenever it takes shape, is
the same engine over a bigger repository with stricter gates, not a new system.

## 2. Distribution semantics are git's semantics

- **Full local replica:** every participant can hold the entire chronicle they
  belong to and operate on it offline.
- **Sync is peer exchange:** remotes, push/pull, fork/PR. Any node can be a
  remote for any other. Hosting (GitHub today) is an interchangeable
  convenience, exactly as it is for git itself.
- **Trust is provenance, not position:** authority comes from attribution and
  evidence carried in the data (ADR-0001's evidence-first principle,
  [PRD-0015](../product/PRD-0015-chronicle-activity-schema-and-open-ingestion.md)'s
  envelope provenance) — never from which server something lives on.
- **Conflict resolution is merge + coherence protocol, not a central arbiter:**
  contradictions are resolved by the PRD-0009 gate and human/agent review at
  the receiving repository, the same way at every tier.

## 3. Services may be conveniences, never dependencies

Indexes, discovery, hosted sync, embedding caches, and coherence bots are
permitted — as **optional accelerators a node can lose without losing
function**. The relationship to any such service must be the relationship git
has to GitHub.

**The exit test:** any participant can clone their data out, point at a
different remote (or none), and retain every core capability — capture,
synthesis, query, promotion. If losing a service breaks a core capability,
that service has become a center, and the design violates this ADR.

## 4. The design test for every future feature

Every new PRD and ADR must be able to answer yes to all three:

1. **Does it work with no server?** (offline, local-only — composes with
   [PRD-0016](../product/PRD-0016-offline-on-device-intelligence.md))
2. **Does it work at N=2?** (two peers exchanging data directly, no third party)
3. **Does the largest scale use the same code path as the personal scale?**
   (policy may differ; machinery may not)

# Consequences

**Positive:**

- **Resilience and longevity:** no single service whose death takes the
  knowledge with it. A chronicle outlives any host, vendor, or company — the
  durability principle (ADR-0001) extended to infrastructure.
- **Privacy floor:** personal chronicles never _have_ to transit anyone's
  server. Combined with local inference (PRD-0016), fully sovereign operation
  is possible — decisive for regulated orgs.
- **Adoption path:** an org can run entirely on infrastructure it already
  trusts (its own git hosting). No new data processor to approve.
- **The global chronicle is derisked:** it becomes an extreme instance of an
  already-proven shape (big repo, strict gates, many forks) rather than a new
  centralized product with new failure modes.
- **Composes with the open boundary work:** PRD-0015's schema/envelope is the
  interchange format between peers; ingestion, promotion, and federation all
  move the same validated data shape.

**Negative / costs:**

- **Eventual consistency is the only consistency.** Two forks of an org
  chronicle can diverge; convergence happens at sync + coherence-gate time,
  not instantly. Features requiring a globally consistent view (e.g. global
  dedup, "is this fact current everywhere?") must be designed as
  per-repository checks that improve with sync frequency.
- **Discovery and search need per-node indexes** or optional index services
  (rebuildable from the repo — the repo is always the source of truth).
  Cheaper to centralize; this ADR forbids depending on it.
- **Identity and trust get harder at community scale.** Self-declared
  provenance suffices inside a trust boundary (PRD-0015's stance); crossing
  boundaries eventually needs signed provenance (commit signing exists;
  envelope signing is anticipated in PRD-0015 §8).
- **Discipline cost:** the three-question design test adds friction to every
  future PRD. That friction is the point.

# Scope

Binds all Rosetta repos, all tiers of chronicle, and all future PRDs/ADRs.
Extends ADR-0001 (adds decentralization to the effective design principles),
ADR-0002 (the engine/data split is the mechanism that makes this possible),
and ADR-0004 (the engine stays machinery-only, so no deployment of it is
privileged). PRD-0009 (coherence), PRD-0015 (open ingestion), and PRD-0016
(offline intelligence) are the first designs written under this constraint;
all three already pass the design test.
