---
id: PRD-0027-experiments
title: Path experiments
date: 2026-08-16
prd: PRD-0027
---

# Path experiments

Small, falsifiable tests. Success is demonstrated value, not quantity of
material retained or connected. No persistent-capture prototype is required
for Experiments 1–3 or the language test. Paper, markdown, or a one-sitting
worksheet is enough.

Do not run Experiment 4 (correction/forgetting controls) as a *product*
prototype until [privacy-and-forgetting.md](privacy-and-forgetting.md) is
reviewed.

## Shared rules

- **N is small.** 5–8 is enough for Experiment 1’s first round; 6–12 is
  enough later to change a hypothesis. Neither claims a category.
- **Length-match conditions** so a “path win” cannot mean “they got more
  text.” The manipulated variable is representation, not information volume.
- **No AI-generated autobiography** as a condition. If a model helps assemble
  a path, the person must see and edit it first.
- **Agents do not invent experiment artifacts alone.** Artifact construction
  is part of the design; see Experiment 1 and
  [materials/README.md](materials/README.md).
- **Record failure modes**, not only successes: confusion, cognitive cost,
  preference without better reconstruction, nostalgia, overhead.
- **Return format:** raw responses first; interpretation only after
  collection. Then: what people understood, what helped, what felt expensive
  or invasive, what did not matter, which brief hypothesis moved.

## Experiment 1 — Path vs. destination (first-round protocol)

**Status:** Protocol ready. Artifacts not yet created. Do not recruit until
Artifact A and Artifact B are reviewed for comparable information.

**Tests:** H2 (curation), H3 (transfer), part of H1 (missing primitive). Also
tests whether the distinction is detectable without teaching the hypothesis.

**Research question.** Does explicitly preserving the path by which a
decision was reached help someone who was not present understand and continue
the work better than a destination-oriented representation containing
approximately the same information?

The first round does not prove the Path hypothesis. It asks whether there is
enough signal to justify deeper investigation.

### Comparable information

Artifacts A and B must contain approximately the same underlying information
and be reasonably similar in length. If B is substantially richer, better
performance does not show that paths matter.

| Artifact | Representation | Same underlying content |
| -------- | -------------- | ----------------------- |
| **A — Destination-oriented** | Conventional summary | Problem, context, alternatives, rationale, decision, outcome |
| **B — Path-oriented** | Explicit relationships and transitions | The same facts, arranged so movement is legible |

Artifact B makes trajectory visible without adding large amounts of new
content. A useful shape (labels are for designers, not participants):

```text
problem
    ↓
question
    ↓
hypothesis
    ↓
alternatives
    ↓
experiment
    ↓
evidence
    ↓
contradiction / uncertainty
    ↓
revision
    ↓
decision
    ↓
outcome
```

Both artifacts describe one real-world decision. Review them together before
recruitment and record the review in
[materials/README.md](materials/README.md). Do not optimize either artifact
for the expected result.

### Division of responsibility

- **This file** defines and preserves the protocol.
- **Experimental materials** (Artifact A, Artifact B, assignment log) are
  developed separately and linked from `materials/` when ready.
- The researcher and conversational research partner construct and review
  the artifacts collaboratively.
- Rosetta agents must not independently generate alternative artifacts
  without preserving the experimental rationale.

### Participants

- **N:** about 5–8.
- Recruit **individually**, not by public solicitation.
- Technical expertise is not required. Prefer mixed backgrounds: technical
  and nontechnical; people who know the researcher well and less well;
  people used to critical thinking or complex decisions.
- One participant may be the reader who independently named the Path
  concept. Do not tell them that is why they were asked.
- **Do not reveal the Path hypothesis** or use the words path, trajectory,
  topology, Rosetta, knowledge graph, or personal chronicle before they
  finish — unless those words already appear in the assigned artifact.
  Independent wording is data.

### Delivery

Run the first round **asynchronously** (Discord, Slack, Messenger, SMS, or
equivalent). A live interview is not required. Async reduces coaching and
lets people sit with the artifact.

Assign each participant **A or B** (balance as evenly as N allows). Do not
show the other artifact until their answers to questions 1–5 are recorded.

### Instructions (send as written)

> I'm testing whether different ways of documenting the reasoning behind a
> decision affect how well someone who wasn't involved can understand it and
> pick up the work afterward.
>
> There is no right answer. Please base your response only on the material
> provided.

Then send only the assigned artifact and the questions below.

### Task (answer from the artifact only)

1. **Reconstruction.** What happened, in your own words?
2. **Reasoning.** Why do you think the team/person made the decision they
   ultimately made?
3. **Uncertainty.** What assumptions, uncertainties, or unresolved questions
   do you think remain?
4. **Continuation.** Imagine you joined this work tomorrow and had never
   been involved in the original decision. What would you want to
   investigate or do next?
5. **Preference.** If you were actually inheriting this work, would you
   rather receive this kind of documentation or the alternative version?
   Why?

For question 5, name the alternative only as “another write-up of the same
decision, arranged differently.” Do not show it and do not call it a path.

**Optional, after 1–5 are in:** If you had to take over this work tomorrow,
what information would you want that wasn’t included here? Do not steer.

**Optional later round:** without reopening the artifact — What do you
remember about the decision and why it was made? Do not add this until the
initial protocol has been used once.

### Researcher behavior

Do not explain the artifact, the hypothesis, or Rosetta during the task.
If asked to clarify:

> Give me your best interpretation based only on what you were given. I'm
> specifically interested in what the artifact allows you to infer without
> additional explanation.

Do not correct misunderstandings during the task. Do not praise answers that
seem to support the hypothesis or react against those that contradict it.
Confusion, disagreement, and rejection of the path-oriented write-up are
data.

### What to record (no formal statistics)

Preserve **raw replies** before interpreting. Then mark, lightly:

- reconstructed the decision
- identified why it was made
- named important assumptions
- named rejected alternatives
- named unresolved uncertainty
- proposed a plausible next action
- confidence seemed calibrated
- preferred the artifact they received
- said they were missing something
- what confused or surprised them
- relationships they noticed on their own
- described something like a “path” without being prompted

### How to read the first round

Discovery, not confirmation.

| Observation | Reading |
| ----------- | ------- |
| A is more useful | Valuable evidence; do not rescue B |
| B preferred but reconstruction is no better | Preference ≠ understanding |
| Both useful for different jobs | Ask which job; do not collapse them |
| B is confusing or expensive | Record cognitive cost |
| No detectable difference | Protocol or hypothesis may not be worth scaling yet |

Intended output of this round:

1. whether the A/B distinction is detectable
2. which aspects of trajectory people actually use
3. whether the idea is understandable without explanation
4. whether the protocol is worth scaling
5. unexpected observations that refine or falsify the current hypothesis

### After artifacts exist

1. Recruit 5–8 people individually and assign A or B.
2. Run asynchronously. Preserve raw responses. Do not interpret mid-stream.
3. Bring the raw set back. Compare to the frontier hypotheses.
4. Decide whether a second, broader experiment is warranted.

## Experiment 2 — Personal re-entry

**Tests:** H2, H5, the Journey-Is-the-Destination candidate (narrow form).

**Setup.** Each participant picks a prior project, decision, or period they
have not touched recently. In one sitting they make a *short self-curated
path* (one page / 5–9 waypoints): experience or problem → interpretation →
challenge → revision or choice → outcome. They then attempt to re-enter the
work for 20–30 minutes using only that path plus whatever they already have
(notes, repo, calendar).

**Compare against their own baseline memory** before they write the path:
write “what I remember now” first, then curate, then re-enter.

**Observe.** Time to first useful action; repeated work avoided; context
surfaced that the pre-path note missed; overhead and nostalgia; whether they
would keep the path.

**Disconfirmation.** The path adds no re-entry value over the pre-path note;
or people report the exercise as homework they would not repeat.

## Experiment 3 — Knowledge transfer (second wave)

**Tests:** H3, H6 (if one handoff is personal and one is work).

**Setup.** One person hands off a project, practice, or decision to another
using (a) a conventional summary and (b) a concise path, order
counterbalanced, similar length. Compare comprehension, time to productive
action, and the recipient’s ability to explain tradeoffs.

Run this after Experiment 1 shows a signal. If Experiment 1 is flat, do not
scale transfer until the packet design changes.

**Disconfirmation.** Recipients act equally well from the summary; path
recipients overfit the narrator’s story and miss still-open options.

## Deferred until the privacy gate

### Experiment 4 — Human correction and forgetting

Prototype *on paper or in a disposable mock* first: correction, suppression,
deletion, expiration, contextual annotation. Test whether people understand
the controls, trust them, and can predict their effects.

Do not attach this to a live chronicle import.

### Experiment 5 — Path granularity

Already folded into Experiment 1’s B′ follow-up. Keep it named so later
rounds can isolate it.

### Experiment 6 — Failure-mode study

Deliberately examine: changed beliefs, false provenance, incomplete history,
private context, coercive sharing, emotionally loaded memories. Define
guardrails before optimizing engagement. Can begin as a tabletop review of
Experiment 1–2 artifacts.

## First wave

1. Finish Artifact A and Artifact B collaboratively; review for comparable
   information; link them from [materials/](materials/README.md).
2. Run **Experiment 1** asynchronously with 5–8 individually recruited
   people.
3. Keep **Experiment 2** and the [language test](language-test.md) queued
   until Experiment 1 shows the protocol is usable.
4. Write raw results, then interpretation, before adding product surface.
