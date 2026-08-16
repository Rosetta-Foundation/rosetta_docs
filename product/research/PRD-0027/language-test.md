---
id: PRD-0027-language-test
title: Path language test
date: 2026-08-16
prd: PRD-0027
---

# Language test

Ask independent readers to distinguish **path**, **timeline**, **history**,
**graph**, and **story** after seeing concrete examples. This answers Feedback
2 (category confusion) without forcing Personal Chronicle into an existing box.

## Protocol

1. Show the five examples below, unlabeled or labeled only as Example A–E.
2. Ask the reader to name each example using at most one of: path, timeline,
   history, graph, story. They may say “none” or “more than one.”
3. Ask: “Which of these would help future-you understand *how you changed your
   mind*?”
4. Ask: “Which of these feels closest to a journal, a second brain, a CRM, a
   knowledge graph, a feed, or an AI memory store — and where does that
   comparison break?”
5. Do not teach the Rosetta definition first. Teaching after the test is
   allowed.

**N:** 5–10 readers who have not been in the PRD conversation.

**Success (weak):** readers spontaneously group Example C with “how it
changed,” not with “what happened in order.”  
**Disconfirmation:** readers cannot tell C from A or D, or they treat all five
as synonyms for “notes.”

## Examples

Use this same decision so differences are structural, not topical.

**Shared endpoint:** “We chose a local-first personal chronicle. Sharing is
deliberate.”

### Example A — Timeline

```text
2026-07-21  Drafted personal vs organizational split
2026-07-31  Wrote founding context
2026-08-02  Sketched voice-log capture
2026-08-16  Named Personal Chronicle
```

Ordered events. No rejected alternatives. No “why.”

### Example B — History

```text
In August 2026 the project accepted a Personal Chronicle PRD. The work grew
out of earlier notes about private versus shared knowledge. The accepted
document now defines personal activity, provenance, and human correction.
```

A retrospective account of what is now true. The path is flattened into a
finished narrative.

### Example C — Path

```text
Experience: a ChatGPT export held more than conclusions — also uncertainty
and later revisions.
    ↓
Initial interpretation: “ingest this as more activity.”
    ↓ challenges
Observation: flattening conversations into session titles keeps destinations
and loses how a belief changed.
    ↓ prompts
Reflection: the next traveler is sometimes future-me.
    ↓ revises
Choice: Personal Chronicle preserves selected relationships, not every
thought; AI suggestions stay non-authoritative.
    ↓ produces
Outcome: a draft PRD that treats path links as meaning, not metadata.
```

Sequence *and* relationship: challenge, revision, rejection, choice. The
endpoint is intelligible because the abandoned reading is still visible.

### Example D — Graph

```text
[ChatGPT export] --imported-as--> [conversation artifact]
[conversation artifact] --evidence-for--> [personal activity]
[personal activity] --related-to--> [engineering activity]
[PRD-0027] --cites--> [ADR-0002]
[PRD-0027] --cites--> [story ch. 7]
```

Typed links among nodes. Useful. Does not by itself say what was tried,
rejected, or later reinterpreted, or in what order attention moved.

### Example E — Story

```text
I used to think a journal was enough. Then I noticed I could remember the
lesson and not the road. Naming that gap felt like turning the civilizational
story inward: leave a path, because I am a traveler too.
```

A telling. It can carry meaning and still be hard to correct, query, or
hand off without rewriting the whole.

## Scoring notes (for the facilitator, not the reader)

| If the reader says… | Signal |
| ------------------- | ------ |
| C is “just a story” | Path vs story is not yet legible; tighten the example, do not add jargon |
| C is “just a timeline with extra words” | Relationship types are not landing |
| D is “the same as a path” | Need a case where edges exist and the journey is still lost |
| “This is a second brain” for all five | Category pull is strong; keep the primitive, add one contrast sentence |
| C would help future-me; A would not | Independent recognition, same family as Feedback 1 |

Record answers verbatim in a dated subsection below when the test is run.
