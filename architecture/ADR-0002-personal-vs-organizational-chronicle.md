# ADR-0002: Personal Chronicle vs. Organizational Chronicle

**Status:** Proposed

**Date:** 2026-07-21

---

> _Private by default. Shared by intention._

---

# Background

Chronicle began as a personal engineering journal intended to automate daily accomplishment tracking, quarterly reviews, and promotion evidence.

As the vision expanded, Chronicle became the foundation for Rosetta's organizational knowledge platform.

This raised an important architectural question:

> How can Chronicle simultaneously support deeply personal career information while also becoming an enterprise knowledge platform?

The answer is to separate **personal knowledge** from **organizational knowledge** — not by forking the engine, but by running one neutral engine against separate data repositories.

---

# Guiding Principle

Every engineer owns their own story.

Organizations own organizational knowledge.

Chronicle should support both without confusing the two.

---

# Decision

Chronicle is one **neutral engine** deployed against many **data repositories**.

The engine captures, enriches, and structures engineering activity. It holds no data of its own and has no opinion about who may read what.

A **data repository** is a store of events. Each repository is either personal or organizational. The engine treats them identically — the repository _is_ the deployment context.

This means:

- `rosetta_chronicle` is the **engine** — machinery only, published as a package and consumed everywhere.
- A personal store (e.g. `rosetta_chronicle_jordan-lee_example`) is a **private data repository** that depends on the engine.
- An organizational store (e.g. `rosetta_chronicle_example-org`) is **another data repository** that depends on the same engine.

There is exactly one engine. Personal and organizational Chronicles are not forks and not feature flags — they are the same machinery pointed at different repositories.

Every store, personal or organizational, is _engine + a repository of events_. Nothing is special-cased.

---

# Personal Chronicle

Every engineer may have their own private data repository.

This repository belongs to the engineer.

Examples include:

- Daily accomplishments
- Career goals
- Promotion strategy
- Performance reviews
- Personal reflections
- Draft architecture ideas
- AI conversations
- Meeting notes
- Learning journal

Chronicle should optimize for helping an individual engineer think, remember, and grow.

This is the engineer's second brain.

---

# Organizational Chronicle

The organization maintains a separate data repository containing only organizational knowledge.

This repository intentionally excludes private career information.

Examples include:

- Architecture Decisions
- Business Rules
- Engineering Standards
- System Documentation
- Technical Investigations
- Incident Reports
- Operational Runbooks
- Engineering Playbooks
- AI Prompt Libraries
- Reusable Workflows

The Organizational Chronicle becomes one of the primary knowledge sources powering Wayfinder.

---

# Privacy Model

Personal Chronicle provides **email-grade privacy**.

This is the same posture people already trust for personal email and private git
repositories:

- Under **normal circumstances**, no colleague or teammate can read a personal store. This is genuine, enforced privacy — the promise that matters day to day.
- Under **exceptional circumstances** (lawful request, account recovery, explicit owner consent), access may be broader — exactly as with email and hosted git today.

Chronicle does not promise more privacy than a private inbox, and does not promise less.

Two obligations follow from this model:

1. **The engine must earn the comparison.** Email-grade privacy is enforced by mature access controls. Chronicle's access model must be equally disciplined: per-person isolation, no organization-wide query that can sweep personal stores, and audit logging on any privileged access. The comparison to email is a promise the engine must be engineered up to — it is not inherited for free.

2. **The owner is responsible for the truly secret category.** Anything that must remain inaccessible under _all_ circumstances should not live only in a synced cloud or org-visible system. Chronicle provides email-grade privacy; it is not an anonymity tool.

> Chronicle provides email-grade privacy for personal knowledge. Owners are responsible for keeping truly secret material out of systems they do not fully control.

---

# Chronicle Core

Chronicle itself remains neutral.

Its responsibility is:

- Capture
- Enrich
- Structure

It should not decide:

- Visibility
- Publication
- Audience

Those are concerns of the consuming deployment. The engine does not know whether it is running against a personal or organizational repository, and it must never behave differently based on that.

---

# Publication

Publication is the seam where an event moves from a personal repository into a
shared repository (team, organizational, community, or global — ADR-0005). The
historical name “organizational” in this ADR still applies; multi-destination
selection and shared-chronicle provisioning are productized in
[PRD-0019](../product/PRD-0019-shared-chronicle-provisioning-and-multi-destination-promotion.md).

It has three properties:

**Drafting is automatic. Publishing is intentional.**
Chronicle may automatically propose a sanitized shared-knowledge candidate from
personal activity — so the engineer never re-types their work. But the
candidate never becomes shared knowledge until a human reviews and approves it
(and chooses destination(s)). Continuous capture; intentional release.

**Provenance re-anchors to shareable evidence.**
A published shared event must trace to artifacts appropriate to that
destination — the pull request, the commit, the incident record, the meeting —
never to the private note that triggered it. The personal note is the _trigger_.
The evidence must stand on its own and be independently shareable.

**Personal knowledge never becomes shared knowledge automatically.**
There is no background sync and no silent fan-out to destinations the human did
not select. The gate is always a deliberate human action. One intentional
promote may land in multiple destinations; each destination applies its own
coherence gate ([PRD-0009](../product/PRD-0009-coherence-protocol.md)).

---

# Example

> The names and identifiers below are fictional and illustrative — used only to
> demonstrate how sensitive career content is filtered out during publication.

Personal Chronicle event:

```yaml
Worked with Sam Rivera on Entra/Okta federation.

Thought this approach was significantly cleaner than the alternatives.

Potential Principal-level architecture discussion.

Would like to reference this during promotion discussions next year.
```

Published organizational event:

```yaml
Architecture Decision

Title:
External authentication federation using Entra and Okta

Participants:
Jordan Lee
Sam Rivera

Outcome:
Selected architecture supports external engineering teams while minimizing
operational complexity.

Evidence:
- PR example-org/auth-gateway#482
- ADR-0007 (rosetta_chronicle_example-org)
- Incident INC-2291 (prior federation outage that motivated the work)
```

Notice that:

- The technical knowledge is preserved.
- Personal thoughts remain private.
- Promotion strategy remains private.
- The organizational event traces to organizational evidence — not to the private note.
- Organizational knowledge becomes reusable.

---

# Relationship

```text
             Personal Chronicle Repository
                    │
     ┌──────────────┼──────────────┐
     │                              │
 Career Growth              Automatic Draft
 Promotion Packets                  │
 Personal Notes                     ▼
 (email-grade private)     Human Review + Sanitize
                                    │
                                    ▼
                       Re-anchor to Org Evidence
                                    │
                                    ▼
                   Organizational Chronicle Repository
                                    │
                                    ▼
                               Wayfinder

        (one neutral engine runs against every repository)
```

---

# Distribution

The hazard to avoid is **shared distribution of a fixed list** — no engineer should ever clone another engineer's log. The workspace bootstrap clones the same repositories for everyone, which is correct for the engine and for shared organizational stores, but would be wrong for per-user private repositories.

Personal Chronicle provisioning therefore must be **user-derived, never list-derived**: the repository is resolved from the _currently authenticated user_, so each engineer only ever creates or clones **their own**. Because the target is the caller's identity — not an entry in a shared config — provisioning it as a **default step of setup is safe**, and is the current behavior. The repository is created **private** (not the organization's `internal` default) and is idempotent: an existing repository is skipped, never overwritten.

This is a deliberate refinement of an earlier framing that called for a separate opt-in command. The original concern was never "provisioning by default" — it was "distributing a shared list of private repos." A user-derived default satisfies that concern directly: there is no shared list, and no engineer's setup can reach another engineer's store.

---

# Benefits

This architecture allows engineers to:

- Keep personal career information private, at email-grade.
- Build promotion evidence continuously.
- Maintain a personal engineering journal.

While simultaneously allowing organizations to:

- Preserve institutional knowledge.
- Improve onboarding.
- Feed AI systems.
- Generate documentation.
- Build organizational memory.

Neither repository replaces the other. They serve different purposes, over one shared engine.

---

# Future Possibilities

Chronicle may eventually support configurable publishing workflows:

- Personal → Draft Organizational
- Personal → ADR
- Personal → Documentation
- Personal → Wayfinder

Each step allows the engineer to review and sanitize before information becomes organizational knowledge.

Publication is always intentional.

---

# Open Questions

**Privacy review — recommended before personal stores hold sensitive career content.**
Confirm the email-grade privacy model with whoever owns data-handling decisions for the deployment: personal engineering notes are practically private, with the same exceptional-access caveats as email. Tracked as a dependency, not a blocker on drafting this ADR.

---

# Philosophy

Personal Chronicle answers:

> "What did I do?"

Organizational Chronicle answers:

> "What should the organization remember?"

Those are different questions.

Chronicle should respect that distinction — with one engine, and two repositories.
