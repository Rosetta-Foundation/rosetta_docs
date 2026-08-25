---
id: chronicle-build-charter
title: Chronicle Build Charter
status: Living
date: 2026-08-25
authorship: collaborative
---

# Chronicle Build Charter

Ready Room exploration is not the decision store. This charter is.

It transfers **invariants, authority, method, and the current envelope** so
Rosetta Agents can continue Chronicle work without shuttling every move
through another conversational agent.

It is **not** a redesign of Chronicle, not a Capture Engine spec, and not
permission to imitate any person. A proposed noun in a prompt is not
authorization to create that noun.

**Session start:** read this file and the private current-state checkpoint
(path below). Continue from the next authorized item. Stop only at an
escalation boundary or a falsifying result.

Private operational state (locators, hashes, vault paths) lives **outside
git**:

```text
~/.local/share/rosetta/chronicle-build/CURRENT-STATE.md
```

Do not copy that file into public repositories. This charter must contain
no private source content.

---

## Engineering invariants

1. Raw evidence and interpretation are different things.
2. Preserve source-native evidence before deriving meaning.
3. Later interpretation must remain temporally later.
4. Filesystem time is not authored event time.
5. Sequence is not causation.
6. Missing provenance must remain missing, not be narratively completed.
7. Uncertainty is data.
8. Every derived or access artifact must terminate in evidence.
9. Private use permission is not public publication permission.
10. Personal data is private by default (ADR-0002).
11. Capture scope is allowlisted, never “collect everything then filter.”
12. Prefer extending existing machinery over introducing ontology.
13. A specimen earns architecture. Architecture does not manufacture specimens.
14. Try to falsify the smallest claim before generalizing it.
15. Rebuildable indexes are not canonical evidence.
16. Current understanding is a view, not a rewritten past.
17. “We don’t know” is a valid Chronicle result.
18. If implementation requires lying about provenance, clocks, or epistemic
    status, stop.

---

## Authority

### GREEN — proceed autonomously

Inspect code, write tests, run bounded specimens, fix bugs, refactor
implementation details, and make reversible naming/layout decisions when
**all** of these hold:

- consistent with the invariants and established decisions below
- no new ontology or schema
- no expansion of private-data scope
- no interpretation or model invocation
- no public disclosure of private source
- reversible
- smallest reasonable change
- backed by evidence or tests

Function names do not require Ready Room approval.

### YELLOW — decision checkpoint; do not implement

Write a short decision memo and stop:

- two plausible architectures with real tradeoffs
- changing an established source contract
- new dependency or storage technology
- changing timestamp semantics
- expanding acquisition scope
- deletion / forgetting / tombstone behavior
- ambiguity: mechanical vs interpretive

### RED — explicit authorization required

- new Chronicle primitive or ontology
- schema migration
- model invocation over private source
- semantic derivation, embeddings, Path, AMP
- broad passive capture; new external account access
- publication or promotion of private findings
- weakening provenance or privacy invariants
- irreversible deletion
- crossing an established epistemic boundary

---

## Method

```text
OBSERVATION → HYPOTHESIS → EXISTING MACHINERY → SMALLEST FALSIFIER
    → SPECIMEN → RESULT → DECISION → BUILD
```

Not: cool idea → TypeScript.

If Russ or a prompt proposes a new subsystem, primitive, field, or
mechanism, first determine whether existing machinery already represents
the requirement. Prefer killing unnecessary abstraction.

After each meaningful cycle, update the **private** current-state file:

- current decisions and status (candidate / validated / rejected)
- evidence / specimens (no public leak of source bodies)
- open hypotheses
- rejected approaches and why
- current authorization envelope
- next falsifier
- escalations requiring Russ / Ready Room

---

## Established decisions

| Decision | Status | Why |
| -------- | ------ | --- |
| Dual track: Daily Chronicle `Activity` vs personal source graph | Validated | Do not route personal raw source through `Activity` |
| ChatGPT inventory then stripped source-graph import | Validated | Topology and clocks; no titles/parts in the graph |
| Source bytes stay outside the public engine | Validated | ADR-0002; Phase 2 graph is not an archive backup |
| Content hash is captured-artifact identity | Validated | Same exact bytes → one object; source-native IDs may identify source structure but do not replace content identity |
| `importedAt` / capture time ≠ source event time | Validated | Do not backdate derivation or evaluation |
| Source vault (copy of observed bytes under operator control) | Validated: locator-loss resolve, one Cursor specimen | Graph without vault cannot resolve after locator loss; not a new engine type |
| Capture / Capture Engine / numbered “levels” as schema | **Rejected** | Existing vault + graph + resolve; do not mint a parallel ontology |
| `getActivity` Claude/Cursor adapters as raw capture | **Rejected** | They summarize and can promote filesystem time to event time |
| Daemon, SQLite, FTS, Gate 4, models, bulk capture | **Not authorized** | Yellow/red until a specimen and explicit envelope say otherwise |

---

## Current authorization envelope

**Last GREEN result (sanitized):** VAULT-V0 passed. One allowlisted Cursor
JSONL: exact-byte SHA-256; copy-if-new; two observations / one object;
resolve after locator hide; source restored. No engine schema, no
interpretation, no public locators.

Source vault status: **Validated: locator-loss resolve, one Cursor specimen.**
Not a Capture primitive. Not bulk capture.

**Authorized next (GREEN, not started):** a synthetic-fixture test that
`SourceContentRepository.resolve` succeeds when `exportPath` is a
same-bytes copy of the existing ChatGPT fixture at a different path.
No API change. No private source.

**Not authorized:** daemon; bulk capture; SQLite/FTS; Gate 4; models;
new primitive; changing source-identity or clock semantics.

**Escalate (YELLOW) if** the next engine step needs a new vault
repository type or a change to hash identity.

---

## Open questions (do not “solve” by inventing types)

- How does Gate 4 deletion interact with vault re-observation?
- Encryption-at-rest / backup / key loss for the vault
- When, if ever, a rebuildable SQLite index is justified
- Whether `SourceContentRepository` should accept a vault path as
  `exportPath` for ChatGPT (existing hash-match resolve may already
  suffice if the vault object is the same bytes)

---

## Ready Room vs Agents

Bring to Russ / Ready Room: invariant changes, YELLOW/RED checkpoints,
surprising specimen results, philosophical wrongness.

Do not bring to Ready Room: ordinary GREEN implementation details.
