# Vision

> This document describes _where Rosetta is going_ — the journey knowledge takes
> through the platform. For _why Rosetta exists_ and the principles it will not
> compromise, see the constitution in [`foundations/`](../foundations/README.md);
> the engineering-era origin story is [FOUNDATIONS.md](FOUNDATIONS.md).

---

Rosetta is built on a simple belief:

Engineering organizations should never lose understanding simply because the work is finished.

Every engineer should have a trusted place to think.

Every organization should have a trusted memory.

These are different needs.

Rosetta intentionally supports both.

---

## Personal Knowledge

Chronicle helps engineers build their own engineering memory.

Not just for performance reviews.

Not just for promotions.

But to become more thoughtful engineers over time.

An engineer's Chronicle should feel like a trusted notebook that quietly remembers everything they would otherwise forget.

---

## Organizational Knowledge

Organizations should not depend on tribal knowledge.

Architecture.

Business rules.

Investigations.

Operational experience.

These should accumulate naturally from engineering activity rather than requiring manual documentation.

Wayfinder exists to make that collective understanding discoverable.

---

## The Flow

Knowledge is born personal.

Over time, some of it becomes organizational.

Eventually, the most durable knowledge becomes institutional.

Rosetta exists to support that journey.

```text
Personal Understanding
          │
          ▼
 Personal Chronicle
          │
          ▼
Intentional Publication
          │
          ▼
Organizational Chronicle
          │
          ▼
      Wayfinder
          │
          ▼
Collective Organizational Memory
```

Not everything should be published.

Not everything should remain private.

Rosetta helps engineers decide what belongs where — _private by default, shared by intention._

The mechanism behind this flow is defined in
[ADR-0002: Personal Chronicle vs. Organizational Chronicle](../architecture/ADR-0002-personal-vs-organizational-chronicle.md).

---

## The Long-Term Goal

The long-term goal is not simply better documentation.

It is to create organizations that continuously learn from themselves.

Every day.

Automatically.

Without asking engineers to do more work.

That is the future Rosetta is designed to enable.
