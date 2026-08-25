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
(path below). Continue the authorized **program**, not one isolated
specimen. Do not stop merely because a specimen passed. Update private
current-state, choose the next smallest GREEN falsifier, and continue.
Stop only at a genuine YELLOW/RED decision, a falsified assumption, or
the program completion condition.

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

Do not return to Ready Room after every successful specimen.

### YELLOW — decision checkpoint; do not implement

Write a short decision memo and stop **that decision**. Other GREEN
work on the program may continue if it does not depend on the answer.

The memo must contain: the decision required; evidence that made it
necessary; viable alternatives; tradeoffs; a recommendation; what
changes depending on the answer. Do not merely ask what to do.

Typical YELLOW:

- two plausible architectures with real tradeoffs
- changing an established source contract
- new dependency or storage technology
- changing timestamp semantics
- expanding acquisition scope
- implementing deletion / forgetting / tombstone behavior before
  semantics are resolved
- implementing encryption or a new custody backend
- ambiguity: mechanical vs interpretive

### RED — explicit authorization required

- new Chronicle primitive or ontology
- schema migration
- model invocation over private source
- semantic derivation, embeddings, Path, AMP
- broad passive capture; new external account access
- publication or promotion of private findings
- weakening provenance or privacy invariants
- irreversible deletion of operator-controlled evidence without a
  resolved forget policy
- crossing an established epistemic boundary
- interpretation, day-view, E8, or broad personal-data capture

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

After each meaningful cycle, update the **private** current-state file
so a fresh agent can determine:

```text
what is established
what evidence established it
what was rejected and why
what remains uncertain
what is currently authorized
what the next falsifier is
what requires escalation
```

Public documentation must remain sanitized. Private locators, source
bodies, sensitive fingerprints, credentials, and reconstructable
personal details do not belong in public research.

---

## Established decisions

| Decision | Status | Why |
| -------- | ------ | --- |
| Dual track: do not route personal raw source through `Activity` | Validated | Conversations, graphs, and vault objects are not a day's `Activity[]` |
| v0.1 Daily Chronicle `Activity` / `getActivity` as the source contract | **Deprecated (frozen)** | Capture and day-view were collapsed. Code remains. Do not extend. |
| ChatGPT inventory then stripped source-graph import | Validated | Topology and clocks; no titles/parts in the graph |
| Source bytes stay outside the public engine | Validated | ADR-0002; Phase 2 graph is not an archive backup |
| Content hash is captured-artifact identity | Validated | Same exact bytes → one object; source-native IDs do not replace content identity |
| `importedAt` / capture time ≠ source event time | Validated | Do not backdate derivation or evaluation |
| Source vault (copy of observed bytes under operator control) | Validated: locator-loss (Cursor file + ChatGPT synthetic directory) | Graph without vault cannot resolve after locator loss; not a new engine type |
| Vault object IDs as `exportPath` | **Rejected** | Reconstruct a same-named directory; HIDE-COPY-V0 |
| Capture / Capture Engine / numbered “levels” as schema | **Rejected** | Existing vault + graph + resolve |
| `getActivity` Claude/Cursor adapters as raw capture | **Rejected** (frozen with v0.1) | They summarize and can promote filesystem time |
| Git-tracked SHA-256 of forgotten bytes as a tombstone | **Rejected** (design) | A permanent hash is a fingerprint of material the operator asked to forget |
| Generalized daemon, SQLite/FTS, embeddings, day-view, E8, Path, AMP | **Not authorized** | Outside this program’s completion condition |
| Broad personal-data capture | **Not authorized** | Allowlist only; no live ChatGPT export in this program |

Do not inflate vault-path findings beyond the specimens that earned them.

---

## Raw-preservation objective

Make Chronicle capable of continuously preserving **explicitly
authorized** raw source material as durable, private,
provenance-preserving evidence, **without interpretation**.

Do not invent a Capture Engine.

```text
explicitly authorized external source
        ↓
observe exact source/native bytes
        ↓
hash
        ↓
private source vault
        ↓
minimal observation/provenance receipt
        ↓
source graph where structural topology matters
        ↓
resolve exact evidence

------------ RAW-PRESERVATION BOUNDARY ------------

interpretation / derivation / evaluation / day-view
```

---

## Authorized program (raw source preservation)

This is a **bounded engineering program**, not one specimen. Design
notes: [`../product/research/PRD-0027/raw-source-preservation.md`](../product/research/PRD-0027/raw-source-preservation.md).
Privacy floor: [`../product/research/PRD-0027/privacy-and-forgetting.md`](../product/research/PRD-0027/privacy-and-forgetting.md).

**v0.1 Daily Chronicle / `Activity`:** remains deprecated (frozen).

### Program A — Gate 4 (design/review first)

Authorized to determine the smallest honest semantics for STOP
observing, WITHHOLD, FORGET captured content, FORGET source-native
identity, and FORGET scope. These are not automatically the same
operation. Chronicle must never claim deletion of data it does not
control.

Do **not** implement Gate 4 until semantics are resolved. If
materially different policies remain, escalate (YELLOW).

### Program B — vault security and key custody (design/review first)

“Vault” is a storage **role**, not a product. Distinguish object
custody from key custody. Evaluate local OS encryption, app-level
encryption, secrets-manager key custody, restic/borg-class tooling,
remote object storage, and secrets-manager-as-object-store (likely
reject/constrain the last).

Do not design custom cryptography. Do not implement encryption unless
an existing mechanism makes a tiny GREEN specimen clearly appropriate
**after** design review.

### Program C — Git boundary (design/review; document the contract)

Private Chronicle Git vs private source vault vs public engine/docs.
Logical storage contract; do not bind Chronicle to one physical
backend. Evaluate the cost of putting raw bytes in ordinary Git
history.

### Program D — successive observations of a mutable source

GREEN specimen once A/B design permits (Gate 4 may stay unimplemented).
Disposable/restorable fixture; prefer a source shape already
understood. No live user data. No Activity. No interpretation of why
A became B. Source-native identity ≠ content identity. Capture time ≠
authored event time.

### Program E — bounded passive acquisition

After D, if still GREEN: one allowlisted disposable source, one
existing/native trigger, same observe → hash → copy-if-new → receipt
operation. Do not build a generalized daemon first. Do not capture
everything.

### Program F — source families

After one passive path works: inspect what exists; do not implement
every candidate source. No UI scraping unless separately authorized.

### Deferred

SQLite/FTS until structural need is demonstrated. No embeddings or
semantic search in the raw-preservation layer.

---

## Current authorization envelope

**Last GREEN result (sanitized):** SUCCESSIVE-V0. Synthetic mutable
source: observe A; re-observe A is copy-if-new; mutate to B; observe
B; resolve A and B independently. Shared source-native id does not
replace content identity. `capturedAt` is observe time, not authored
event time. No Activity.

Prior: HIDE-COPY-V0 / VAULT-CHATGPT-V0 / RESOLVE-COPY-V0 / VAULT-V0.

**Program status:** Raw-preservation program authorized 2026-08-25.
A/B/C design is in
[`../product/research/PRD-0027/raw-source-preservation.md`](../product/research/PRD-0027/raw-source-preservation.md).
D passed. Existing Chronicle hooks are frozen v0.1 `Activity` and
must not be reused for Program E.

**YELLOW — Ready Room decisions (do not implement past these):**

1. Gate 4 V0: accept **scope-only** forget (recommended) vs require
   byte-level refuse or crypto-shred in V0.
2. Security V0: document **local dir + OS disk encryption**
   (recommended) vs implement Chronicle-owned encryption / remote
   custody now.
3. Program E: no honest existing trigger (hooks are frozen v0.1).
   Adding launchd/fswatch/daemon is new acquisition infrastructure.
   Manual snapshot already is the shared observe operation.

**Next GREEN that does not depend on those answers:** none that
advances the milestone. Do not start E, Gate 4 implementation, or
encryption until the matching YELLOW is resolved.

**Not authorized in this program:** interpretation; day-view; E8;
Path; AMP; broad personal-data capture; live ChatGPT export; new
Capture type; object-id resolve; generalized daemon; SQLite/FTS;
embeddings; implementing Gate 4 or encryption before the YELLOW
decisions in the design note are resolved.

---

## Completion condition

Continue until one of these is true:

1. a genuine YELLOW/RED decision is reached that blocks the next
   dependent step;
2. a core assumption is falsified;
3. the path reaches this bounded milestone:

```text
private durable vault
+ honest observation receipts/provenance
+ operator STOP/forget semantics
+ appropriate V0 security boundary
+ successive mutable-source preservation
+ one bounded passive acquisition path
```

Then return a **program checkpoint**, not merely the last test result:

- what is demonstrated
- what remains design-only
- what remains unimplemented
- security/privacy limitations
- evidence/specimens
- architectural changes actually made
- rejected abstractions
- next recommended program
- Ready Room decisions required

---

## Ready Room vs Agents

Bring to Russ / Ready Room: invariant changes, YELLOW/RED checkpoints,
surprising specimen results, philosophical wrongness.

Do not bring to Ready Room: ordinary GREEN implementation details, or
“the last specimen passed.”
