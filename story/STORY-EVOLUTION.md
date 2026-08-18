# Story Evolution — Three altitudes, one thesis

**Status:** Working memo (not a manuscript rewrite)
**Date:** 2026-08-18
**Source:** Captured agent response, 2026-08-18. The argument is preserved
here so the implementation can teach the philosophy without hijacking it.

This memo tests newer engineering understanding against Chapters 1–10. It
does **not** authorize new chapters, a product pitch in the essay voice, or
software types in the manuscript.

The ten-chapter structure stays. The README already says further chapters
should exist only when a genuinely new civilizational idea remains after
Chapter 10. That discipline still holds.

---

## What to protect

The existing manuscript already does something easy to lose and hard to
recover: **it makes the reader arrive at Rosetta before naming Rosetta.**

Protect that fiercely.

The stated constraint is that Rosetta appears as consequence rather than
pitch, and that the manuscript should remain true even if the project
disappears. Chapter 10 still culminates correctly:

> A practice eventually seeks instruments → Rosetta named as one possible
> expression.

What has changed is not that constraint. What has changed is that **we now
understand the thing more concretely than when that story architecture was
designed.**

Do **not** rewrite the story around the software architecture. No
`TransformationExecution` in Chapter 7. The architecture's job, for the
essay, is better ordinary-language examples — not a glossary of engine
types.

---

## Three communication altitudes

Not “simple docs” and “technical docs.” Three artifacts serving three
different cognitive jobs.

| Altitude | Audience | Question | Sketch |
| --- | --- | --- | --- |
| **The Story** | educated general reader, ~10th-grade readability | *Why should this exist?* | Chapters 1–10; one-page restatement in [`../guides/ROSETTA-IN-ONE-PAGE.md`](../guides/ROSETTA-IN-ONE-PAGE.md) |
| **The Field Guide** | builders, civic leaders, journalists, knowledge workers | *What does this let us do?* | [`../guides/WHAT-ROSETTA-MAKES-POSSIBLE.md`](../guides/WHAT-ROSETTA-MAKES-POSSIBLE.md) |
| **The Architecture** | senior engineers, architects, researchers | *How can we trust that it actually does it?* | [`../architecture/CONCEPTUAL-MODEL.md`](../architecture/CONCEPTUAL-MODEL.md) |

They should describe **the same ideas at different resolutions**, not become
three competing explanations.

ADRs remain the wrong home for the third altitude. ADRs answer *why did we
choose this implementation?* The conceptual model answers *what is the
system?*

---

## 1. The Story: civilization keeps destinations and loses paths

Chapters 4–9 already move from information abundance, to lost organizational
context, to relationships between ideas, to lost paths, to selective
path-keeping.

Recent engine work has sharpened what a **path** actually means.

It is no longer merely:

> Bob proposed X, Jane objected, the team learned Y, therefore we chose Z.

It is becoming:

> **What existed? What did we observe? What did we make of it? What changed
> because of that interpretation? And can someone later reconstruct that
> honestly?**

That is a stronger idea. It should enter the story as horsies-and-doggies,
not as types.

Property taxes may be one of those examples:

> A homeowner receives a tax bill. The number survived. But did the path
> survive?

Then walk backward in ordinary language:

```text
Your bill changed
      ↑
tax rates changed
      ↑
budgets were adopted
      ↑
proposals were debated
      ↑
staff made recommendations
      ↑
conditions and evidence produced those recommendations
```

Now “the path is the data” is not software philosophy.

It is **civic literacy**.

---

## 2. The Field Guide: build from questions

The missing middle artifact is not another README.

It is a question-driven guide: *what does this let us do?*

Examples already strong enough to sketch:

- **Why did my property taxes go up?** Wayfinder could trace the
  assessment, taxing jurisdictions, rate changes, budget decisions,
  hearings, votes, and available explanations.
- **Why does our system work this way?** Chronicle could reconstruct the
  decisions, incidents, alternatives, rejected approaches, and evidence
  that produced the current architecture.
- **Where did this belief come from?** A personal Chronicle could trace a
  reflection back through earlier conversations, observations, documents,
  and interpretations.
- **Why can't you answer?** Because part of the path is missing. And
  Rosetta should tell you **what is missing rather than pretending to
  know.**

That last question is becoming one of Rosetta's most distinctive ideas.

The Field Guide can be highly accessible. Diagrams. Concrete examples.
Minimal jargon. It must not invent shipped civic products. “Could” is the
honest verb until a domain is built.

---

## 3. The Conceptual Architecture: the same questions, less abstraction

Give the architect the exact same questions, then remove the abstraction.

For **Where did this belief come from?** show:

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

Then explain the invariants:

**Identity. Immutability. Provenance. Event time versus ingestion time.
Reconstructed versus contemporaneous capture. Partial knowledge. Boundary
between source and interpretation. Personal versus organizational
context.**

And crucially:

> A provenance graph is not merely metadata about the Chronicle.
>
> **It is part of the Chronicle.**

That is senior-architect territory. It belongs in
`architecture/CONCEPTUAL-MODEL.md`, not in an ADR and not in Chapter 7.

---

## The deeper change in the story

“Memory” may no longer be sufficient as the central metaphor.

Preserving things is not the hardest problem.

**Preserving relationships between things is.**

And even that is not quite enough.

We need relationships between:

**experience → evidence → interpretation → decision → consequence.**

That is why the engine work keeps refusing to become a database of
memories. Every time the work moves forward, another distinction has to
survive:

```text
source ≠ interpretation

event time ≠ ingestion time

definition ≠ execution

execution ≠ result

missing ≠ nonexistent

unresolved ≠ false

human interpretation ≠ machine interpretation

private memory ≠ organizational memory
```

A deeper formulation of the existing thesis:

> Civilization became extraordinarily good at preserving **what we
> concluded**.
>
> It remains surprisingly bad at preserving **how we came to conclude it**.

That may be the intellectual hinge around Chapters 5–7.

Not necessarily a rewrite. More like: **the engineering work has supplied
evidence that the existing thesis goes deeper than we originally
understood.**

Then the civic expansion is the same question at a larger scale:

| Scale | Question |
| --- | --- |
| Personal | Why did *I* come to believe this? |
| Organizational | Why did *we* make this decision? |
| Civic | Why did *our government* do this? |
| Civilizational | Why did *we inherit the world this way?* |

**Same question. Different scale.**

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

Chronicle preserves enough of the path to investigate those questions.

Wayfinder lets a human ask them.

Rosetta is the broader practice and infrastructure that makes the exchange
possible.

This diagram may eventually become one of the most important in the
corpus. It does not yet belong in the essay. Chapter 10 already names the
instrument late, as consequence. The scale diagram is a later illustration
of an idea the chapters have already earned.

---

## Chapter-by-chapter test

The test is whether the newer formulation *deepens* a chapter or *displaces*
it. Displacement would be a rewrite. Deepening can wait.

| Chapter | Existing core | Newer formulation | Verdict |
| --- | --- | --- | --- |
| 1 | Shared learning; civilization begins when experience becomes collective | Personal → civilizational is the same question at different scale | Deepens the eventual illustration; do not add the scale diagram here |
| 2 | External memory; knowledge becomes persistent | Preserving things is not the hardest problem | Compatible. Ch. 2 earns persistence; later chapters earn paths |
| 3 | Personal bridge into the Information Age | Still the narrator's crossing, not the thesis hinge | Leave it. Do not load civic or engine language here |
| 4 | Information abundance ≠ understanding | Destinations accumulate faster than routes | Already pointed at the later hinge. No rewrite |
| 5 | Organizations lose continuity before they lose information | Relationships between experience, evidence, interpretation, decision, and consequence | **Hinge candidate.** The chapter already says people carry relationships documents do not. The newer chain is a sharper name for that fragility |
| 6 | Humans navigate relationships between ideas | Wayfinder is how a person asks; do not name it here | Compatible. Keep tools as emerging possibility, not product |
| 7 | Destinations survive more readily than paths | The stronger path: what existed, what we observed, what we made of it, what changed, and whether a later mind can reconstruct that honestly | **Hinge candidate.** Supply civic / ordinary examples later. Do not import engine types |
| 8 | Abundance does not automatically become inheritance | Civic literacy: a tax bill can survive while its path does not | Deepens the obligation. A property-tax walk-back would fit here or in Ch. 7 *if* a revision is ever opened |
| 9 | Selective path-keeping; disagreement; hospitality across time | Missing ≠ nonexistent; say what cannot be answered | Compatible. “Why can't you answer?” is a practice requirement, not a feature list |
| 10 | A practice seeks instruments; Rosetta named late | Memory may be too small a metaphor; relationships and provenance are the harder inheritance | Still correct. Do not add a Chapter 11. If the metaphor shifts, it belongs as a later revision of 7–10, not a new civilizational idea |

**Conclusion of the test:** the ten-chapter structure still holds. The
newer work does not introduce a new civilizational idea after Chapter 10.
It supplies evidence that Chapters 5–7 were already pointing at something
deeper: not only lost context, and not only lost paths between ideas, but
lost honesty about how a conclusion was reached — including the honesty of
saying what is missing.

A later real-corpus smoke test sharpened a 10th-grade form of that same
honesty. The system encountered a missing attachment and did not invent
it. That line belongs in the communication artifacts, not as a new
chapter:

> **Good memory isn't remembering everything.**
> **Good memory also means knowing what you've forgotten.**

See [`../guides/ROSETTA-IN-ONE-PAGE.md`](../guides/ROSETTA-IN-ONE-PAGE.md)
and the checkpoint in
[`../product/research/PRD-0027/provenance-checkpoint.md`](../product/research/PRD-0027/provenance-checkpoint.md).
The engine has now proved it can preserve and traverse a path honestly
enough to ask the *next* question — can it derive meaning without
pretending the meaning was in the source? — without rewriting the essay
to announce that result.

---

## What this memo does not do

- It does not rewrite Chapters 1–10.
- It does not add a Chapter 11.
- It does not move software types into the essay.
- It does not treat Civic Blueprint, property-tax tracing, or any
  particular domain product as shipped.
- It does not accept a new metaphor by proclamation. “Memory may no longer
  be sufficient” is a **working hypothesis** to test against the manuscript,
  not a founding rewrite of [`../foundations/MANIFESTO.md`](../foundations/MANIFESTO.md).

Claim type for the hinge sentence (*civilization keeps conclusions better
than the journeys that produced them*): **I** (interpretive), confidence
**Working hypothesis**, basis the existing Ch. 5–7 thesis plus engine
distinctions that keep refusing to collapse. See
[`../process/epistemic-status.md`](../process/epistemic-status.md).

---

## Related

- Essay: [`README.md`](README.md), [`WRITING-CONTEXT.md`](WRITING-CONTEXT.md)
- Path research: [`../product/research/PRD-0027/path-research-brief.md`](../product/research/PRD-0027/path-research-brief.md)
- Engine design (implementation, not story):
  `rosetta_chronicle/docs/design/derived-records.md`,
  `rosetta_chronicle/docs/design/transformation-registry.md`
