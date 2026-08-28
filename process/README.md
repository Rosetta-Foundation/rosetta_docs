# Process — claim hygiene for Rosetta writing

Protocols for **evidentiary discipline** in essays, foundations, PRDs, and ADRs:
how we source claims, label confidence, challenge weak assertions, and test
historical patterns.

These are **not** the SDLC evidence gates (PRD-0011). Different jobs:

| Surface                   | Question                                                 | Machinery                                     |
| ------------------------- | -------------------------------------------------------- | --------------------------------------------- |
| **SDLC / Chronicle**      | Did the _implementation_ meet the spec, with provenance? | `rosetta_dev-scripts/sdlc-workflow`, ADR-0007 |
| **Process (this folder)** | Do the _claims in our writing_ earn their confidence?    | Protocols below                               |

**Status:** Proposed (2026-08-01). Adapted for Rosetta from Civic Blueprint’s
research / adversarial / historical-parallel stack
(`civic-blueprint/project-2028/agent/process/`), stripped of civic-governance
specifics. Portable ideas only; not a fork of that corpus.

## Protocols

| Doc                                                | Question it answers                                                            |
| -------------------------------------------------- | ------------------------------------------------------------------------------ |
| [`research-protocol.md`](research-protocol.md)     | What evidence do we have, at what source weight, with what citation integrity? |
| [`epistemic-status.md`](epistemic-status.md)       | How confident are we, and what would change that?                              |
| [`adversarial-review.md`](adversarial-review.md)   | Does the claim set survive structured challenge (preferably cross-lineage)?    |
| [`historical-parallel.md`](historical-parallel.md) | Have structurally similar patterns appeared before — including failures?       |
| [`chronicle-build-charter.md`](chronicle-build-charter.md) | Chronicle implementation doctrine: invariants, GREEN/YELLOW/RED, method, envelope |
| [`chronicle-v1-readiness-2026-08-28.md`](chronicle-v1-readiness-2026-08-28.md) | Sanitized V1 readiness: allowlisted export observe + source-graph catalog; Desktop locate is next |

## Claim types (quick reference)

Full detail in [`epistemic-status.md`](epistemic-status.md):

| Code  | Type                      | Typical instrument                                 |
| ----- | ------------------------- | -------------------------------------------------- |
| **H** | Historical / empirical    | Research Protocol                                  |
| **P** | Pattern / mechanism       | Historical Parallel (+ Research)                   |
| **I** | Interpretive thesis       | Adversarial Review + epistemic table               |
| **O** | Practitioner observation  | Research W6; do not upgrade to universal law alone |
| **M** | Memoir / phenomenological | Non-evidence N1 unless asserted as public fact     |
| **N** | Normative / value         | No empirical verifier — cite as commitment         |
| **D** | Definitional / analytic   | Low research load; watch smuggled empirics         |

## Where this is used today

- Story claim inventory: [`../story/CLAIMS-CATALOG.md`](../story/CLAIMS-CATALOG.md)
- Early-access challenge form: [`.github/ISSUE_TEMPLATE/challenge-claim.yml`](../.github/ISSUE_TEMPLATE/challenge-claim.yml)

## Provenance labels (drafting)

When an artifact’s authorship matters for trust, label it in frontmatter or a
short note:

| Label                           | Meaning                                               |
| ------------------------------- | ----------------------------------------------------- |
| `human`                         | Written by a human without material AI drafting       |
| `collaborative`                 | Human directed; AI assisted; human owns the text      |
| `ai-generated, steward-curated` | AI drafted; human reviewed and accepts responsibility |
| `ai-generated`                  | Uncurated — not for load-bearing claims               |

## Relationship to foundations

[`../foundations/PRINCIPLES.md`](../foundations/PRINCIPLES.md) — **Evidence
First** and **Source Driven** are the values these protocols operationalize for
_prose_, the same way SDLC operationalizes them for _ships_.
