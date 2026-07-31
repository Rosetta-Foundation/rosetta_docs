# Principles

**Status:** Founding Context (v0.1)

**Date:** 2026-07-31

The values and design philosophy every Rosetta decision is evaluated against.
[`CONTEXT.md`](CONTEXT.md) explains why these exist;
[`DECISIONS.md`](DECISIONS.md) records where they have been applied and made
binding.

---

## Founding Values

Every decision — product, architectural, or organizational — should answer
these questions with a consistent "yes":

1. Does this improve stewardship?
2. Does this preserve knowledge?
3. Does this reduce unnecessary friction?
4. Does this help people coordinate?
5. Does this increase understanding?
6. Does this empower contributors instead of replacing them?
7. Will future generations thank us for this decision?

---

## Design Principles

### Private by Default, Shared by Intention

Knowledge is born personal. What becomes part of any collective memory is the
author's decision, made deliberately — never by default, never automatically.
This is a privacy stance and a respect-for-authorship stance; it is what makes
shared memory trustworthy enough to build on.

### Evidence First

Generated knowledge must always be traceable back to evidence. Trust is earned
through provenance — attribution carried in the data, never authority derived
from where something is hosted.

### Source Driven

Never ask a contributor to duplicate work they have already done. Derive
understanding from activity; documentation is a byproduct, context is the
product.

### Decentralized by Construction

Every chronicle — personal, team, organizational, community, global — is the
same machinery over a distributed repository. No tier requires a central
server. Services may be conveniences, never dependencies. Anyone can clone
their knowledge out and retain every core capability
([ADR-0005](../architecture/ADR-0005-decentralized-by-construction.md)).

### Durable

Knowledge should improve over time and survive reorganizations, promotions,
resignations, vendors, hosts, and changing AI models. Institutional memory
outlives the tools that created it.

### Extensible & Open

Structured knowledge is exposed at open boundaries — formal schemas, push
ingestion, pluggable inference — so systems we never anticipated can produce
into and consume from the platform.

### AI Native, Human Centered

AI is a first-class producer and consumer of knowledge, not a feature. And the
platform exists for people: AI increases human capability, never replaces
human judgment. Neither humans nor AI are secondary consumers.

---

## Preferences in Design

When making product decisions, prefer systems that are:

- transparent
- composable
- inspectable
- interoperable
- open
- extensible
- explainable
- collaborative
- resilient
- durable

Avoid unnecessary complexity. Optimize for understanding rather than novelty.
If a technological decision serves today's implementation but undermines the
long-term philosophy, the philosophy wins.

---

## Success Is Measured By

- fewer ideas lost
- less duplicated effort
- contributors finding relevant context quickly
- communities preserving institutional memory
- people building upon previous work instead of restarting
- easier collaboration across organizations and disciplines
- knowledge improving across generations

Never by user counts, AI feature counts, or engagement metrics.
