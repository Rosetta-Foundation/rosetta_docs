---
id: PRD-0027-provenance-checkpoint
title: Provenance checkpoint — real corpus, honest path
date: 2026-08-18
prd: PRD-0027
status: Working record
source: Captured agent response, 2026-08-18
---

# Provenance checkpoint

The real private corpus hit the machinery and the architecture held.

This file records that checkpoint. It does **not** contain the corpus. It
does not authorize summarizing ChatGPT conversations. It does not widen
[PRD-0027](../../PRD-0027-personal-chronicle.md).

## Historical status

This file records the frontier **when E3 had been measured and E4 was
still next** (2026-08-18). It is not a description of the engine after
E4–E6. Do not retrofit those later measurements into the progression
or “next experiment” sections below.

Later measured work is documented elsewhere:

- E4 / E4b — `rosetta_chronicle/docs/design/interpretation-policy.md`
- E5 — `rosetta_chronicle/docs/design/evaluation.md`
- E6 — `rosetta_chronicle/docs/design/current-understanding.md`
- Architect-altitude sync after those measurements —
  [`../../../architecture/CONCEPTUAL-MODEL.md`](../../../architecture/CONCEPTUAL-MODEL.md)

The human path-vs-destination trials in [`experiments.md`](experiments.md)
remain a separate track. Do not renumber them. The sequence below is the
**engine progression** — what the provenance machinery has now proved, and
the next experiment it has earned.

---

## What held

The important part is not merely that commands returned `ok`. The
distinctions Rosetta was designed to keep stayed distinct under real data:

- real source topology survived reconstruction
- multiple cited nodes remained independently traversable
- derived → execution → definition/source lineage worked **backward**
- source node → execution → derived worked **forward**
- removing a definition produced **`partial` rather than fiction**
- restoring it repaired the path
- unrelated corruption stayed isolated
- missing attachments remained missing
- no private corpus material leaked into the public engine

That is a stronger result than another synthetic test suite. The provenance
model survived contact with actual history.

Evidence for subgraph-local integrity was the reason
[rosetta_chronicle#14](https://github.com/Rosetta-Foundation/rosetta_chronicle/pull/14)
was held. The private smoke test validated that concern against the real
corpus, not only the fixture. Merge of that PR is an engine decision; this
file records the docs-side reason it became mergeable.

---

## Specimen B — the attachment boundary

One specimen gave a clean architectural split:

```text
Source Graph:
"This node references an attachment,
and the attachment is missing."

Provenance Graph:
"This interpretation cites this node."
```

The provenance walk currently does **not** say:

```text
"This path contains a missing attachment."
```

That is not a bug. The report is right. The two graphs are answering
different questions.

A trustworthy memory system must still be able to remember:

> **Something was here, and we no longer have it.**

The system encountered a conversation containing a missing artifact and
did **not** invent the missing thing. That sounds trivial. It is not.

---

## Known frontier — artifact-level provenance / attachment lineage

Capture this so it is not forgotten. Do not treat it as the next build.

Source nodes can themselves contain provenance-bearing references.

Today Rosetta could say:

> This decision cites meeting node X.

Eventually we want:

> Meeting node X referenced report Y, but the underlying artifact is
> unavailable.

That is a richer form of epistemic honesty — the same “why can't you
answer?” idea, one layer deeper into the source.

**Not necessarily next.** Just do not forget that a cited node may point
at something the Chronicle does not hold.

---

## Engine progression

```text
E1  Can we preserve the source?
    ✅

E2  Can we preserve the relationships?
    ✅

E3  Can we traverse the relationships honestly?
    ✅

E4  Can a machine interpret the source
    while preserving the distinction between
    source and interpretation?
    ???
```

We have been refusing automatic interpretation until the machine could
answer **Where did this come from?** It now can — not perfectly across
every future artifact type, but enough to begin the next experiment.

```text
PHASE WE JUST PROVED

Can Rosetta preserve and traverse
the path honestly?

           ✅

NEXT EXPERIMENT

Can Rosetta derive meaning
without pretending the meaning
was present in the source?
```

The next PR must **not** be “summarize all ChatGPT conversations.”

---

## Next architectural question — interpretation policy

We already have `TransformationDefinition`, `TransformationExecution`,
and `DerivedRecord`.

The next question is:

> **What is an interpretation policy?**

That is how the first actual machine-produced transformation should
arrive: named, narrow, attributable, and reviewable.

### Narrow first recipe (candidate)

**Extract candidate observations from one conversation.**

Not:

> Tell me what this person learned.

But:

> Given these cited nodes, identify 1–3 candidate observations and
> explicitly distinguish direct evidence from inference.

Every output gets:

- exact source refs
- transformation definition
- model identity
- prompt / version
- execution metadata
- review state `unreviewed`
- **no automatic Chronicle promotion**

That is the first interesting test of **machine interpretation with
provenance**.

Do not silently inherit today's deterministic execution collapse. Current
`TransformationExecution` identity is idempotent and content-addressed:
two observationally identical deterministic runs persist as one artifact
because `createdAt` is not identity-bearing. Artifact identity and
occurrence identity are not the same concept. Whether machine
interpretation needs a distinct execution-occurrence record — especially
for nondeterministic producers — is an open E4 design question, not a
truth already decided by the human-note machinery. See
[`../../../architecture/CONCEPTUAL-MODEL.md`](../../../architecture/CONCEPTUAL-MODEL.md).

Run it first against **Specimen A**, now that the specimen's structure is
known. Keep the private corpus out of the public engine.

### Privacy floor

[`privacy-and-forgetting.md`](privacy-and-forgetting.md) still gates any
live-chronicle import, always-on logging, or model-written biography.
E4 is an interpretation experiment on an already-held source graph. It
does not waive that gate, invent Activity, or promote.

---

## What this file will not do

- Quote or commit private corpus text
- Treat E3 as proof that every future artifact type is honest
- Collapse source graph into provenance graph
- Authorize a summarizer
- Renumber or replace the human trials in [`experiments.md`](experiments.md)

---

## Communication consequence

The missing-attachment result belongs in the story-altitude artifacts as
ordinary language, not as engine types:

> **Good memory isn't remembering everything.**
> **Good memory also means knowing what you've forgotten.**

See [`../../../guides/ROSETTA-IN-ONE-PAGE.md`](../../../guides/ROSETTA-IN-ONE-PAGE.md)
and [`../../../guides/WHAT-ROSETTA-MAKES-POSSIBLE.md`](../../../guides/WHAT-ROSETTA-MAKES-POSSIBLE.md).
The architect-altitude statement of the same idea is the `partial`
result and the attachment-lineage frontier in
[`../../../architecture/CONCEPTUAL-MODEL.md`](../../../architecture/CONCEPTUAL-MODEL.md).
