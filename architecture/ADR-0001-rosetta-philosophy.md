# ADR-0001: Rosetta Philosophy

**Status:** Accepted

**Date:** 2026-07-21

---

# Summary

Rosetta exists to solve a fundamental problem in modern software engineering:

> Valuable engineering knowledge is constantly created, but very little of it survives.

Commits explain *what* changed.

Jira explains *what was requested.*

Documentation explains *what should be true.*

AI conversations explain *how we reasoned.*

Slack explains *why decisions were made.*

Architecture diagrams explain *how systems fit together.*

None of these, individually, preserve organizational understanding.

Rosetta exists to connect them.

---

# Vision

Rosetta is an AI-native engineering knowledge platform.

Rather than asking engineers to produce more documentation, Rosetta continuously transforms the work they are already doing into durable organizational knowledge.

Its purpose is not documentation.

Its purpose is organizational memory.

---

# Core Philosophy

Documentation is a byproduct.

Context is the product.

Every engineering artifact contributes context:

- Git
- GitHub
- Jira
- Claude Code
- Architecture
- Pull Requests
- Slack
- Confluence
- Design Documents
- Incidents
- Meetings

Rosetta continuously assembles those fragments into coherent understanding.

---

# Architecture

Rosetta is intentionally divided into separate responsibilities.

## Rosetta

Mission.

The overall platform responsible for engineering memory.

---

## Chronicle

Memory.

Captures engineering activity and converts it into structured knowledge.

Chronicle should never become tightly coupled to a single user interface.

It is the source of truth.

---

## Wayfinder

Understanding.

Consumes Chronicle to help engineers navigate systems, discover knowledge, understand business rules, and answer architectural questions.

Wayfinder is not "better documentation."

Wayfinder helps engineers understand systems.

---

# Design Principles

## AI Native

AI is not an add-on.

AI is a first-class consumer of organizational knowledge.

---

## Evidence First

Generated knowledge should always be traceable back to evidence.

Trust comes from provenance.

---

## Human and AI

Knowledge should be equally valuable to:

- engineers
- managers
- future AI systems

---

## Continuous

Knowledge should accumulate automatically.

Engineers should spend less time documenting and more time engineering.

---

## Durable

Engineering decisions should remain understandable years later.

Institutional memory should survive reorganizations, promotions, and employee turnover.

---

# Long-Term Mission

Rosetta should eventually answer questions like:

> Why does this code exist?

> What business problem led to this architecture?

> What production incident caused this behavior?

> Who investigated this issue?

> What alternatives were considered?

> What systems depend on this component?

Those answers should emerge naturally from engineering activity rather than requiring manual documentation.

---

# Brand Philosophy

The platform is named **Rosetta** because its purpose is translation.

The Rosetta Stone made ancient languages understandable by relating multiple representations of the same knowledge.

Rosetta applies the same principle to engineering.

It translates between:

- Human understanding
- Source code
- Business intent
- Organizational knowledge
- Artificial intelligence

The platform does not create knowledge.

It reveals the relationships already hidden within it.

---

# Visual Identity

## Rosetta

Represents translation and understanding.

The proposed icon should consist of:

- three interlocking geometric stone facets
- a luminous gold center representing shared understanding
- subtle circuit traces emerging from the base
- strong symmetry
- timeless vector design

The logo intentionally combines:

Ancient permanence
+
Modern engineering
+
Artificial intelligence

Unlike Wayfinder, whose gold element points outward toward direction, Rosetta's gold element should remain at the center.

Wayfinder answers:

> "Where should I go?"

Rosetta answers:

> "How does it all fit together?"

Chronicle answers:

> "What happened?"

Together they represent navigation, memory, and understanding.

---

# Guiding Principle

Every commit tells part of the story.

Rosetta remembers the rest.
