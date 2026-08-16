---
id: PRD-0027-landscape
title: Path landscape research
date: 2026-08-16
prd: PRD-0027
status: First pass
---

# Landscape research

**This is landscape research, not design guidance.** Adjacent products show
what existing primitives already solve and where they break. They do not
dictate Rosetta’s destination.

**Pass:** first inventory (2026-08-16). Sources are mostly **W5/W6**
(product docs, practitioner comparison). Not a T2 sweep. AI synthesis in this
file is **W7** and is not evidence.

For each cluster, the questions are:

1. What is the primary object?
2. How is time represented?
3. Is the relationship between things first-class or incidental?
4. Who owns, edits, and deletes the record?
5. What does the system optimize for?
6. What important aspect of path does it lose?

## Comparison

| Cluster | Representative objects | Time | Relationship | Ownership / delete | Optimizes for | Path aspect typically lost |
| ------- | ---------------------- | ---- | ------------ | ------------------ | ------------- | -------------------------- |
| **PKM / linked notes** (Obsidian, Roam, Logseq) | Note (file) or block (bullet) | Daily notes; mtime; optional journals | Bidirectional links and backlinks are first-class; *why the link exists* and *what was rejected* are incidental | Local-first files in the strong cases; owner edits and deletes notes | Capture, linking, retrieval | Revisable trajectory; challenge/revision as meaning; selective forgetting as a first-class act rather than “delete the note” |
| **Journaling / lifelog / reflection** | Entry, prompt, streak | Calendar day or session | Usually sequential; later rereading is a feature, not a typed relation | Owner; apps vary from local to hosted | Expression, habit, sometimes insight prompts | Distinguishing destination from path; provenance to source evidence; non-authoritative machine reflection |
| **Personal CRM / relationship memory** | Person, interaction, reminder | Last-contact and follow-up | Person–event links; social graph | Often hosted; org-flavored sharing | Coordination and recall of *who* | Interior change of mind; rejected interpretations; owner-as-future-traveler rather than pipeline |
| **AI-assistant memory** (ChatGPT memory, Mem0-class layers) | Extracted fact / preference / “memory” | Recency; session vs persistent profile | Implicit: retrieval similarity, sometimes a memory graph | Vendor or app-mediated; update/delete APIs exist (e.g. Mem0 delete-by-id / scoped delete; ChatGPT memory settings) | Prediction and personalization; token reduction | Human-authored path; append-only correction that keeps the old reading visible; “do not infer who I am” |
| **Knowledge graphs, event sourcing, provenance, VCS** | Node/edge, event, commit, triple | Event time, valid time, commit history | Edges and commits are first-class; RDF-star / reification can annotate edges | Depends; git is owner-controlled history that is hard to truly forget | Integrity, lineage, reconstruction of *state* | Humane forgetting; emotional meaning; curation as intelligence rather than complete log replay |
| **Timelines, feeds, quantified self** | Event, metric, post | Strict chronology or rank | Co-occurrence and “also happened”; rarely “challenged / revised” | Platform or device | Capture, engagement, monitoring | Meaning of movement; permission to leave things unrecorded |
| **Oral history, archives, narrative therapy, learning science** | Testimony, fonds, session, worked example | Life course, accession, instructional sequence | Provenance and finding aids; therapeutic or pedagogical relation | Consent-heavy in the good cases; institutional in archives | Witness, care, transfer of understanding | Everyday personal agency at software speed; risk of over-fitting “story” as the only object |

### Notes on the PKM cluster

Obsidian’s public position is local, private, open-format notes with
user-made links ([obsidian.md](https://obsidian.md/)). Practitioner
comparisons treat Obsidian as file-primary and Logseq as block-primary, with
Roam as the outliner/graph reference
([MakerStack comparison](https://makerstack.co/compare/logseq-vs-obsidian/),
[Logseq review](https://doolpa.com/article/logseq)). Links are abundant.
A link is not yet a path: it does not have to record sequence of attention,
friction, a rejected reading, or a later reinterpretation.

### Notes on the AI-memory cluster

Mem0 documents an extract-and-retrieve layer: conversations become durable
facts; search returns memories for the next prompt; update and delete exist
for correction and erasure
([how it works](https://docs.mem0.ai/core-concepts/how-it-works),
[delete](https://docs.mem0.ai/core-concepts/memory-operations/delete)).
The stated product move is from “chat with history” to “evolving user
profile.” That is state-optimization. PRD-0027’s correction loop is the
opposite shape: keep the old reading, mark the evaluation, compute current
understanding as a view.

## Working observation (not a conclusion)

No cluster in this pass has **path** as its primary object in the sense of
the [research brief](path-research-brief.md): a selected, attributable,
revisable trajectory whose relationships are the meaning, with humane
forgetting and non-authoritative inference.

That is consistent with H1. It is not proof of H1. A later pass may find a
closer neighbor (research notebooks, ADR practice, Talmudic commentary,
decision records, zettelkasten “trails,” Bush’s associative trails).

## Next landscape pass

- Decision records / ADRs / RFCs as path-keeping in engineering
- Vannevar Bush “associative trails” vs a first-class Path object
- Archival appraisal (what is allowed to fade) as a forgetting analogue
- One hosted journal and one AI-memory product, walked with the six questions
  by a human (not this agent) before citing them as load-bearing
