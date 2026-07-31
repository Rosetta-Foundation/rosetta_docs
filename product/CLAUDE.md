# product

Product Requirements Documents (PRDs) for the Rosetta platform.

This folder holds PRDs — the "what and why" of a capability, written **before** it is built: goals,
non-goals, and acceptance criteria. A PRD is the input to the phased implementation specs and the
build that follows (PRD → phased SDD specs → build).

PRDs are a first-class Rosetta artifact, alongside the ADRs in `../architecture`. Where an ADR
records a decision already made, a PRD frames a capability to be built. Over time Chronicle and
Wayfinder should capture and surface these PRDs as durable "why does this exist" knowledge.

**Legibility is a hard requirement.** Every PRD must be legible to **both humans and machines** —
structured frontmatter, list-based goals/non-goals, checkbox acceptance criteria, explicit links.
The end objective is that agents can author, consume, and act on PRDs, not just people.

Start from `TEMPLATE.md`; register each new PRD in `README.md`.
