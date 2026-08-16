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

- **N is small.** 6–12 participants is enough to change a hypothesis; it is
  not enough to claim a category.
- **Length-match conditions** where possible so “path won” cannot mean “they
  got more text.”
- **No AI-generated autobiography** as a condition. If a model helps assemble
  a path, the person must see and edit it first.
- **Record failure modes**, not only successes: nostalgia, overhead, false
  provenance, pressure to over-record.
- **Return format:** for each experiment, what people understood, what helped,
  what felt invasive, what did not matter, which brief hypothesis moved.

## Experiment 1 — Path versus endpoint recall

**Tests:** H2 (curation), H3 (transfer), part of H1 (missing primitive).

**Setup.** Choose one closed artifact with a real origin path (a short
decision, a small design, a resolved disagreement). Prepare two packets of
similar length:

| Condition | Packet |
| --------- | ------ |
| A — Endpoint | Conclusion or artifact + a conventional summary |
| B — Path | Same endpoint + a curated path: what was tried, what challenged it, what was rejected, what was chosen, what happened next |

Assign participants to A or B. After a delay (same day or next day), ask them
to:

1. Restate the conclusion
2. Rate confidence
3. Explain *why* it was chosen, including at least one rejected alternative
4. Revise the conclusion given one new contrary fact

**Measures.** Recall accuracy; calibration (confidence vs accuracy); quality
of explanation (mentions rejected alternatives and constraints); quality of
revision (updates without discarding still-valid constraints).

**Disconfirmation.** B does not beat A on explanation or revision once length
is matched; or B only wins by smuggling extra facts rather than relationships.

**Minimum useful path (H5 edge).** If B wins, try a shorter B′ (3–5 waypoints)
on a new pair. The target is the minimum meaningful representation.

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

Run **1**, **2**, and the [language test](language-test.md). Stop and write
results here before designing more product surface.
