---
id: PRD-0027-historical-self-sample-2026-08
title: Historical-self sample — 2026-08
date: 2026-08-22
prd: PRD-0027
status: Working record
source: Operator-sealed private experiment, 2026-08; sanitized after analysis
---

# Historical-self sample — 2026-08

Sanitized research record of a completed private experiment on a
historical handwritten personal journal.

It is **not** an engine change, **not** E7 Track A, **not** E8, and
**not** a product requirement. It does not modify Chronicle
architecture, evaluation semantics, current-understanding policy,
schemas, handlers, or CLI. It does not widen
[PRD-0027](../../PRD-0027-personal-chronicle.md).

The Revision Provenance Hypothesis in
[thought-evolution-2026-08-19.md](thought-evolution-2026-08-19.md)
remains **Speculative**.

Private material is not in this repository: photographs, transcripts,
journal wording, names, visible dates, unit-level content, and
anything that could reconstruct the journal.

Type: **H** (sampling and sealed table, as reported) and **I**
(semantic candidate). AI wording in the originating analysis is
**W7** and is never evidence.

---

## Purpose

Sample historical pages independently of present-day agreement, then
ask whether these questions can yield different answers on the same
later interpretation:

1. What does the source support? (textual evidence support)
2. Does the operator now judge the interpretation accurate of
   **historical-self**? (2026 judgment about the historical state)
3. Does the operator recognize that same interpretation as describing
   **present-self**? (2026 judgment about now)

The purpose was **not** to find five mind-changes, write a biography,
or manufacture a genuine E7 revision.

---

## Sampling (page-side is the unit)

| Item | Recorded |
| ---- | -------- |
| Population | N = 15 eligible written page-sides |
| Eligibility | Visible handwritten marks; census without reading for meaning |
| Precommitted reveal order | 9, 5, 1, 10, 6 |
| Usable / replacements | Five usable; no replacements |
| Blind before reveal | Operator had not read the selected pages for meaning |
| Adjacent pages | Not added for context (split or thin entries stayed in-sample) |
| Source time | Journal identity is not an exact timestamp; undated or mixed-date pages keep source-time uncertainty. Visible contemporaneous dates were not published here |

The randomized sampling unit is the **page**. Do not treat
interpretations or bounded units inside a page as independently
sampled observations.

---

## Protocol variants

Two procedures were used. The change was made **before** evaluations
on the later pages were revealed.

**Pages 9 and 5 (original).** Source → stance → 0–3 candidate
interpretations → A → B → C.

**Pages 1, 10, and 6 (adapted).** Bounded textual units /
stance-coded observations, then the same A → B → C.

Where:

- **A** = textual evidence support
- **B** = historical-self accuracy (2026 about historical-self)
- **C** = present-self recognition (2026 about present-self)

B was not a contemporaneous historical evaluation. `evaluatedAt` was
not backdated.

**Operator-as-interpreter.** No independent helper and no model
produced the interpretations. The operator authored stance /
interpretation (or bounded units) and later sealed A/B/C. That limits
independence between interpretation and autobiographical judgment.

---

## Procedural limitations (preserved)

- **Page 1:** fatigue was explicitly reported during interpretation.
  Later sealed judgments were kept. Fatigue is a limit, not a reason
  to delete those answers.
- **Page 5:** the B prompt accidentally referred to another page
  index. Surrounding procedure concerned page 5. The typo is part of
  the record; it is not silently rewritten.

---

## Sealed results (sanitized)

### Page level (random sample of five sides)

- All five pages contained **at least one** interpretation judged
  historically accurate (**B** YES) and **not** presently recognized
  (**C** NO).
- Pages **1** and **10** also contained **at least one**
  interpretation judged historically accurate **and** presently
  recognized (**B** YES and **C** YES).

The sample therefore contains **continuity and discontinuity**. It
does not classify the historical self as wholesale rejected.

### Unit-level totals (descriptive, clustered only)

Across 35 sealed units, clustered in five pages and two protocol
variants:

- textual support: 34 supported, 1 uncertain
- historical-self accuracy: YES × 35
- present-self recognition: NO × 28, YES × 6, cannot assess × 1

These totals are **descriptive only**. Do not report them as a
population rate or as “the operator changed.” Units were not
identically sampled.

---

## Candidate semantic finding

**Evaluation event time is distinct from the historical or present
state (or context) the judgment is about.**

In this run, A/B/C were 2026 acts. **B** and **C** often differed
(YES vs NO) on the same interpretation. Both answers can be
simultaneously valid. That is not a 2017 evaluation, and it is not
by itself a later revision of an earlier recorded recognition.

Existing Chronicle E5/E6 clocks record **when a judgment occurred**
(`evaluatedAt`) and **when it was persisted** (`recordedAt`). They
do not name **which self/time a recognition dimension is about**.

Competing encodings that were considered and **not** adopted here:

- Backdating **B** to the source year would falsify assessment time.
- Two `personalRecognition` values at the same `evaluatedAt` would
  look like `conflict` / `same-evaluator-tie` (one-target
  contradiction).
- Two `personalRecognition` values at different `evaluatedAt` values
  would look like revision (one-target change).
- `evidenceSupport` is textual support, not historical-self accuracy.
- `precedingEvaluationId` is optional “responds to” lineage, not a
  target of recognition.

This note does **not** add a schema, dimension, or handler. Off-engine
scoresheets already held the split. If a later design ever represents
the pattern, it would need to keep assessment time, judged object,
state/context the judgment is about, evaluator, and question type
from collapsing — as a honesty constraint, not as a feature request
in this PR.

Confidence of the semantic candidate: **Working hypothesis** (this
sample; operator-as-interpreter). Not Established. Same-lineage
analysis is an **upper bound** until a human or cross-lineage pass
lands.

---

## Threats to validity

These weaken exciting readings (biography, proven revision, required
schema), not the bare sealed table.

- Operator generated both interpretation and A/B/C. Uniform **B** YES
  may be generation bias (only writing what already seemed
  historically accurate).
- Source genre may make historical/present divergence easy to observe
  (speech-acts that are not automatically present-tense self-claims).
- The journal was already known to be from a historically different
  life period.
- Five pages only; page sample is random, units within pages are not.
- Protocol changed after two pages; later pages contributed more units.
- Fatigue on page 1; prompt typo on page 5.
- Retrospective reconstruction error; 2026 identity may shape **B**.
- Source is stronger evidence of **expressed stance** than of inner
  historical state.

---

## What this is not

- **Not E7 Track A.** There was no contemporaneous historical
  `DerivedEvaluation`, and no genuine T2→T3 sequence recorded through
  Chronicle on an existing machine interpretation.
- **Not evidence that makes E7 greener.** Dual 2026 targets are a
  neighboring question. Do not manufacture a conceptual T2/T3 from
  **B** vs **C**.
- **Not** a proof of the Revision Provenance Hypothesis (remains
  **Speculative**).
- **Not** a model-quality result (no provider).
- **Not** a population claim, an objective truth claim, or a
  biography / profile.
- **Not** a demonstration that Chronicle needs a particular schema.

Engine checkpoints remain where they were:
`rosetta_chronicle/docs/design/revision-experiment.md` (E7),
`evaluation.md` (E5), `current-understanding.md` (E6).

---

## Deferred options (not started in this record)

These remain future options only. This file does not authorize them.

- Candidate A on the existing private E4b corpus (genuine
  `personalRecognition` of an already-committed machine
  interpretation; still not a manufactured revision).
- Helper / second-observer replication if a suitable adult
  participant becomes available (tests whether **B**/**C** still
  diverge when the operator did not author the interpretations).

Do not treat either as a backlog item that an agent should start
from this note.

---

## Privacy

[`privacy-and-forgetting.md`](privacy-and-forgetting.md) still gates
persistent-capture prototypes. This experiment photographed five
sampled sides into a local private hold. It did not import a live
chronicle, invoke a provider on the journal, or commit source.

---

## Epistemic status

| Claim | Type | Confidence | Basis | What would change this |
| ----- | ---- | ---------- | ----- | ---------------------- |
| N=15; order 9, 5, 1, 10, 6; five usable; no replacements; operator blinded before reveal | H | Working hypothesis | Operator sampling report | A contemporaneous protocol log showing a different N, order, or replacement |
| All five pages had at least one B YES / C NO; pages 1 and 10 also had B YES / C YES | H | Working hypothesis | Operator-sealed table (page-level) | Sealed scoresheets showing a page without that pattern |
| Unit totals 34/1, 35, 28/6/1 | H | Working hypothesis | Descriptive clustered count only | Rescored units; not a population estimator |
| Operator authored interpretations (no helper/model) | H | Working hypothesis | Protocol as run | Evidence of an independent interpreter on those pages |
| Evaluation event time ≠ state/context the judgment is about | I | Working hypothesis (upper bound) | B≠C at one 2026 assessment time in this sample | Helper-authored interpretations where B and C never diverge, or a showing that B is only generation bias |
| This is not E7 Track A | D | Normative / analytic | No Chronicle T2→T3; no contemporaneous historical evaluation | An Accepted record that this run *was* logged as Track A (it was not) |
| Revision Provenance Hypothesis | I | Speculative | Unchanged; this run is not that test | See thought-evolution note |

---

## Non-goals of this file

Do not, from this note:

- change E5, E6, or E7 documentation as if the engine grew a new axis
- start E8
- start Candidate A or a helper replication
- invent `derived-evaluation/2` or a recognition-target field
- upgrade the Revision Provenance Hypothesis
- publish or reconstruct the journal
