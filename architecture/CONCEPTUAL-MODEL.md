# Rosetta Conceptual Model

**Status:** Sketch
**Altitude:** The Architecture — *How can we trust that it actually does it?*
**Audience:** senior engineers, architects, researchers
**Date:** 2026-08-18
**Updated:** 2026-08-19 (E4–E6 measured; transformation subgraph unchanged)

This document answers *what is the system?* ADRs answer *why did we choose
this implementation?* Keep those jobs separate.

The Field Guide asks the same questions in ordinary language
([`../guides/WHAT-ROSETTA-MAKES-POSSIBLE.md`](../guides/WHAT-ROSETTA-MAKES-POSSIBLE.md)).
This page removes the abstraction. It is not a license to put these types
into the essay.

Implementation detail lives in the engine design docs. This page states
the conceptual invariants those docs are already protecting.

---

## The question, without the parable

**Where did this conclusion, interpretation, decision, or belief come from?**

```text
                     TransformationDefinition
                              ↑ cites
                              │
SourceNode ← cites ─ TransformationExecution
                              │
                              │ produces
                              ↓
                         DerivedRecord
```

This is the **transformation subgraph**: recipe → run → interpretation
event. It is not the whole Chronicle. Physical provider invocations,
later human evaluations, and current understanding sit beside it, not
on it.

| Record | Owns | Does not own |
| --- | --- | --- |
| **Source node / source graph** | What existed and how it was structured | Meaning, review, “the takeaway” |
| **Transformation definition** | The immutable recipe: what kind of interpretation this is | A particular run or result |
| **Transformation execution** | The durable record of a transformation run under the current identity policy: who, when, configuration, cited sources, output handles | The interpretation body or review state; a complete ledger of every physical invocation |
| **Derived record** | The interpretation event: content, producer, review seed, source refs | The recipe, or a living concept that can be edited in place |

Definitions explain the recipe. Executions record a run under the
engine's current identity policy. Derived records explain the
interpretation.

A later mind reconstructing a conclusion, interpretation, decision, or
belief walks this graph — and, when they exist, the evaluation history
projected over it. That walk *is* the answer to the Field Guide
questions:

| Field Guide question | Conceptual object |
| --- | --- |
| What existed? | Source artifact / source graph / source node |
| What did we observe? | The cited source refs on the execution |
| What did we make of it? | Derived record (interpretation event) |
| What process produced that reading? | Transformation definition + execution |
| What did someone later think about that reading? | Derived evaluation (append-only; not a rewrite of the derived record) |
| What is current for this perspective as of T? | Current-understanding view (computed; not a durable historical artifact) |
| What changed because of it? | Later derived records, decisions, or activities that cite this one |
| Why can't you answer? | A missing node, a missing citation, or a reconstructed gap marked as such |

---

## Provenance is part of the Chronicle

A provenance graph is not merely metadata about the Chronicle.

**It is part of the Chronicle.**

If you can delete the relationships and still claim to have the memory,
you have destinations again. The path did not survive.

That is why the engine keeps refusing to become a database of memories.
The durable object is not only the conclusion. It is the reconstructable
relationship among experience, evidence, interpretation, decision, and
consequence.

---

## Invariants

These distinctions must survive. Collapsing any of them makes the system
easier to demo and harder to trust.

### Identity

The transformation chain has three records and three identities.
Do not collapse definition, execution, and derived record into “the
transformation.” `createdAt` is never part of any of these ids. A living
concept — “my current reading of this conversation” — is none of them.
It would be a later view over a sequence of events.

| Record | Identity hashes | Same payload again | A different id when |
| --- | --- | --- | --- |
| **Definition** | Recipe fields: type, version, description, deterministic flag, allowed producer types | Same artifact (`already-present`; first `createdAt` kept) | Description or flags change, even if type@version matches |
| **Execution** | Definition id, recipe type/version, source refs, producer, configuration, output *content* hashes | Same persisted artifact (`already-present`; first `createdAt` kept) — not a second occurrence record | Those inputs change. Derived-record ids are not in this hash |
| **Derived record** | Source refs, type, record-schema version (`derived-record/1`), producer, content hash | Same interpretation event (`already-present`; first `createdAt` kept) | Content, producer, refs, type, or record-schema version change |

A definition is a recipe artifact, not an event. An execution is the
durable record of a transformation run; current deterministic recipes
deduplicate observationally identical runs. It is not the interpretation
body. A derived record is an immutable interpretation event, not a
living conceptual row.

`executionId` on a derived record is a **link**, not part of the derived
id. The same derived event can be written with `record-derived` alone.
Recipe version on an execution (`1`) is a different axis from
`DerivedRecord.transformationVersion` (the record schema,
`derived-record/1`).

Current deterministic execution identity is idempotent and
content-addressed. Re-running an observationally identical deterministic
transformation does not create a second persisted execution artifact
because `createdAt` is not identity-bearing. Therefore
`TransformationExecution` is not a complete ledger of every physical
invocation.

Two clock times can still be one persisted execution if recipe, producer,
refs, configuration, and output hashes match. Two physical model calls
that emit different output hashes are two artifacts.

**Occurrence is a fourth identity, not a fourth belief node.**
`ExecutionOccurrence` is the receipt of one physical provider
invocation: whether the provider answered, and whether Chronicle
accepted that output as memory. It is distinct from
`TransformationExecution`. It is not a provenance-graph node and is
not included in the default provenance walk. Artifact identity and
occurrence identity are not the same concept.

**Narrow machine interpretation is registered.** The
`candidate-observation` recipe is the nondeterministic
(`deterministic: false`) path that produces machine-authored derived
records from explicitly cited source nodes. Caller-supplied
`record-derived` / `transform-record` remain deterministic and reject
that type. Do not inherit deterministic collapse semantics for this
recipe.

**Evaluation is a later historical act, not a rewrite.**
`DerivedEvaluation` is append-only. It cites one derived record.
`evidenceSupport` and `personalRecognition` are independent dimensions:
judging cited evidence sufficient is not personal recognition, and
omitting recognition leaves it unassessed. Identity includes
`evaluatedAt`, so a later judgment is a new artifact. The evaluated
derived record is not mutated.

**Current understanding has no identity in this table.** It is a
read-only recomputable projection over interpretation and evaluation
history for a named perspective and an event-time `asOf`. It is not
another durable historical artifact and not a “latest value” field.
`explanation.evaluationIds` are the in-scope historical evaluations
considered for that record/perspective. Dimension
`contributingEvaluationIds` are only the act(s) responsible for the
current reduced state.

### Immutability

Edits are new records, not in-place rewrites. Correction is append-only.
Review state on an event is a seed, not a license to mutate the event's
meaning. A later evaluation does not rewrite the interpretation it
cites. Recomputing current understanding does not write a new memory
object.

### Provenance

Every derived record must answer **why does this exist?** by citing
sources structurally — not by inference from timestamps or directory
proximity. An execution cites a definition and source nodes. A derived
record may link to the execution that produced it; that link is not a
substitute for source refs. An evaluation cites the derived record it
judges. An occurrence may point at an execution after commit; that
receipt is still not a belief node.

### Event time versus ingestion time

When something happened in the world is not when the engine learned
about it. Inventories already separate `ingestedAt` from event-time
ranges. Reconstruction after the fact must not pretend to be
contemporaneous capture.

### Reconstructed versus contemporaneous capture

A later interpretation of an old source is a derived record. It is not
the source, and it is not a backdated observation. Honesty about *when
the reading happened* is part of the path.

### Partial knowledge

Missing ≠ nonexistent. Unresolved ≠ false. A gap is a first-class
answer. Filling it with plausible prose is a failure of the model, not a
feature of the guide.

A real private-corpus smoke test produced **`partial` rather than
fiction** when a cited definition was removed, and repaired the path
when it was restored. Unrelated corruption stayed isolated. Missing
attachments stayed missing. That is what that smoke showed — not a
general load-test claim, and not a claim that every future artifact
type will behave the same. See
[`../product/research/PRD-0027/provenance-checkpoint.md`](../product/research/PRD-0027/provenance-checkpoint.md).

### Boundary between source and interpretation

```text
source ≠ interpretation
definition ≠ execution
execution ≠ result
execution ≠ occurrence
human interpretation ≠ machine interpretation
evaluation ≠ interpretation
evidence support ≠ personal recognition
historical evaluation ≠ current contributor
current understanding ≠ durable historical artifact
```

Shortcut `source node → Activity` drops branches, uncertainty, context,
and transformation history. Source graphs are not Activity. Derived
records are not Activity. Activity, when it exists, is a Chronicle
*representation* — a destination — and must cite the path that produced
it.

### Personal versus organizational context

```text
private memory ≠ organizational memory
```

Same machinery, different permission. Promotion is intentional, never
automatic ([ADR-0002](ADR-0002-personal-vs-organizational-chronicle.md)).
A personal path is not a smaller org knowledge graph.

---

## Same questions, different scale

The types do not change when the audience does.

```text
PERSON
"What happened to me?"

        ↓

TEAM / ORGANIZATION
"Why did we do this?"

        ↓

COMMUNITY / GOVERNMENT
"Why was this decided?"

        ↓

CIVILIZATION
"How did we get here?"
```

What changes is policy, consent, and which chronicle you are allowed to
read. What must not change is the reconstructability of the path — or
the obligation to say when it cannot be reconstructed.

---

## Known frontier — artifact-level provenance

Source nodes can themselves contain provenance-bearing references.

```text
Source Graph:
"This node references an attachment,
and the attachment is missing."

Provenance Graph:
"This interpretation cites this node."
```

These are different answers. The provenance walk today can say that an
interpretation cites a node. It does not yet say that the path contains
a missing attachment. That is not a bug. It is the next known honesty
layer:

> This decision cites meeting node X.
> Meeting node X referenced report Y, but the underlying artifact is
> unavailable.

Do not collapse the source graph into the provenance graph to fake that
sentence. Capture attachment lineage as a frontier, not as the next
required build.

---

## What this is not

- **Not an ADR.** No implementation choice is being ratified here.
- **Not Activity.** Do not collapse this model into daily chronicle
  synthesis.
- **Not a product pitch.** Civic examples in the Field Guide are
  questions the model could support, not shipped domains.
- **Not the essay.** Keep these names out of Chapters 1–10.

---

## Related

- Story evolution (why this page exists): [`../story/STORY-EVOLUTION.md`](../story/STORY-EVOLUTION.md)
- Provenance checkpoint (E3-era real-corpus evidence; E4 was still next): [`../product/research/PRD-0027/provenance-checkpoint.md`](../product/research/PRD-0027/provenance-checkpoint.md)
- Field Guide: [`../guides/WHAT-ROSETTA-MAKES-POSSIBLE.md`](../guides/WHAT-ROSETTA-MAKES-POSSIBLE.md)
- Engine: `rosetta_chronicle/docs/design/derived-records.md`,
  `rosetta_chronicle/docs/design/transformation-registry.md`,
  `rosetta_chronicle/docs/design/interpretation-policy.md`,
  `rosetta_chronicle/docs/design/evaluation.md`,
  `rosetta_chronicle/docs/design/current-understanding.md`,
  `rosetta_chronicle/docs/architecture.md`
- Decisions already made: [ADR-0001](ADR-0001-rosetta-philosophy.md),
  [ADR-0002](ADR-0002-personal-vs-organizational-chronicle.md)
