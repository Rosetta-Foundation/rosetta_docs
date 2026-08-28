---
id: chronicle-build-charter
title: Chronicle Build Charter
status: Living
date: 2026-08-28
authorship: collaborative
---

# Chronicle Build Charter

Ready Room exploration is not the decision store. This charter is.

It transfers **invariants, authority, method, and the current envelope** so
Rosetta Agents can continue Chronicle work without shuttling every move
through another conversational agent.

It is **not** a Capture Engine spec and not permission to imitate any
person. A proposed noun in a prompt is not authorization to create that
noun.

**Operating posture (2026-08-26):** build the smallest coherent Chronicle
**V1 the operator can turn on and use**, while aggressively protecting
canonical raw evidence, privacy, authorization boundaries, and genuinely
irreversible actions.

Architectural uncertainty alone is not an escalation condition. **Blast
radius and reversibility determine escalation.**

> Protect precious evidence and irreversible boundaries fiercely.
> Experiment aggressively with replaceable code.

> A specimen is allowed to be implementation. Building something small
> is often the fastest way to learn whether the architecture is correct.

**Session start:** read this file and the private current-state checkpoint
(path below). Continue the V1 program. Do not stop because a specimen
passed or because two implementations were plausible. Stop at RED, a
product-intent fork that actually needs the operator, a falsified core
premise, or a V1 readiness review.

Private operational state lives **outside git**:

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

## Evidence classes (treat differently)

```text
CODE
    experimental, replaceable, rollback-friendly

INDEXES / CACHES
    preferably rebuildable, disposable

STRUCTURAL METADATA / SOURCE GRAPHS
    preferably reproducible from canonical evidence
    migrations allowed with verification

RAW SOURCE VAULT
    precious, canonical captured evidence — protect aggressively

PRIVACY / EXTERNAL DISCLOSURE
    potentially irreversible

DELETION / KEY DESTRUCTION
    potentially irreversible
```

Do not protect experimental code with the conservatism used for
irreplaceable personal evidence.

---

## Authority

### GREEN — proceed autonomously

Ordinary implementation and architectural hypotheses that are cheap to
revise. Multiple plausible implementations do **not** make a decision
YELLOW.

Includes: interfaces, repository layout, filesystem layout, naming,
watcher vs polling, JSON/receipt shape that does not invent epistemic
claims, internal APIs, refactors, fixtures, adapter details, helpers,
strategies with tested rollback, anything whose failure is “change the
code and try again.”

Pick the simplest defensible option, record meaningful assumptions,
implement, test, continue.

### YELLOW — consequence, not an automatic STOP

Use sparingly. Changing it later would mean migration or compatibility
work, but would not destroy evidence, expose private data, or become
practically irreversible.

When hit:

1. Determine whether a conservative V1 default exists.
2. Document the assumption and future migration boundary.
3. Prefer the option that preserves future optionality.
4. If still reversible/migratable, **choose the V1 default and continue**.
5. Return to Ready Room only if operator/product intent is genuinely
   required.

### RED — STOP and request explicit authorization

Meaningful blast radius that cannot be cheaply corrected:

- publishing or exposing private source
- sending private corpus to a new external third party
- capturing sources/scopes the operator did not explicitly authorize
- irreversible deletion of the only known copy of canonical raw evidence
- destructive canonical-data migrations without verified rollback
- cryptographic/key operations that can permanently make evidence
  unrecoverable
- weakening a security/privacy boundary around an existing private corpus
- making a private repository public
- irreversible disclosure of credentials
- crossing from raw preservation into semantic/model interpretation when
  that materially changes epistemic claims or privacy exposure
- other actions where `git revert` cannot undo the consequence

---

## Method

Preferred loop:

```text
choose the smallest defensible design
        ↓
checkpoint (git)
        ↓
implement
        ↓
test / falsify
        ↓
learn
        ↓
keep / revise / revert / migrate
        ↓
continue toward V1
```

Do not optimize for discovering final architecture before
implementation. The sunk-cost fallacy is not an architectural invariant.

Prefer small commits and bounded waves.

After meaningful cycles, update the **private** current-state file so a
fresh agent knows: established / evidence / rejected / uncertain /
authorized / next work / RED.

Public docs stay sanitized.

---

## Established decisions

| Decision | Status | Why |
| -------- | ------ | --- |
| Dual track: do not route personal raw source through `Activity` | Validated | Not a day's `Activity[]` |
| v0.1 Daily Chronicle `Activity` / `getActivity` | **Deprecated (frozen)** | Do not extend. Do not use for V1 raw observe |
| ChatGPT inventory then stripped source-graph import | Validated | Topology and clocks; no titles/parts in the graph |
| Source bytes stay outside the public engine | Validated | ADR-0002 |
| Content hash is captured-artifact identity | Validated | Source-native IDs do not replace content identity |
| Capture / observe time ≠ source event time | Validated | Do not invent authored time |
| Source vault (copy-if-new of observed bytes) | Validated on fixtures | Locator-loss + successive states; not a Capture type |
| Vault object IDs as ChatGPT `exportPath` | **Rejected** | Reconstruct a same-named directory |
| Capture Engine / numbered “levels” as schema | **Rejected** | Vault + graph + resolve |
| Git-tracked SHA-256 of forgotten bytes | **Rejected** | Fingerprint of material the operator asked to forget |
| Gate 4 V1: **scope-only forgetting** | **Approved V1 default** | STOP/WITHHOLD/delete what Chronicle controls; no byte-level refuse; no crypto-shred; never claim vendor deletion |
| Security V1: private local vault + OS disk encryption | **Approved V1 default** | No Chronicle-owned app encryption to reach V1; custody abstraction may evolve |
| Program E: bounded passive trigger | **Approved** | Frozen v0.1 hooks must not be reused; a tiny watcher/poller is allowed; no generalized daemon first |
| Broad personal-data capture / live export ingest | **RED until explicit pilot** | Fixtures until a pilot request |
| Day-view, interpretation, E8, Path, AMP, embeddings | **Not V1 raw-preservation** | Do not smuggle intelligence into capture |

---

## V1 product target

Smallest Chronicle the operator can turn on:

```text
configure explicit source scope(s)
→ start
→ authorized source changes
→ observe exact bytes → hash → copy-if-new → receipt
→ STOP / forget-scope
→ resolve retained evidence
→ survive process restart (data on disk; watcher is restarted by the operator)
```

No summaries, classifications, embeddings, personality/emotion/Path/AMP
detection, candidate observations, model-generated metadata, Daily
Chronicle or day-view synthesis, or biography.

**Sources:** smallest useful set that exists honestly on the host.
Cursor JSONL under an allowlisted path, ChatGPT export via existing
path, ordinary allowlisted files. Do not block V1 because Claude Code
data is absent here. No home-directory sweep, no browser scraping, no
capture-everything.

**Passive progression:** manual observe (done) → one bounded automatic
trigger → one allowlisted path → restart/recovery of stored evidence →
second source only if it tests generality → reusable service only if
earned.

**Indexing:** optional, rebuildable, evidence-terminating. Earned by
use, not discussion.

**Backup:** identify what cannot be rebuilt (vault objects). Document
V1 limitation rather than adding a cloud backend. Sending private
source to a new external provider is RED.

**Real personal data:** STOP and request RED authorization before a
pilot. Successful fixtures are not silent broad capture.

---

## Current envelope

**Last GREEN (sanitized):** directory observe on `main` (#31). Prior:
file observe (#30), SUCCESSIVE-V0, VAULT-V0, RESOLVE-COPY-V0,
VAULT-CHATGPT-V0, HIDE-COPY-V0.

**Now:** V1 observe + first RED-authorized ChatGPT **data-export**
pilot (private vault) and stripped source-graph catalog (private
`import-chatgpt`). Sanitized readiness:
[`chronicle-v1-readiness-2026-08-28.md`](chronicle-v1-readiness-2026-08-28.md).
Design notes:
[`../product/research/PRD-0027/raw-source-preservation.md`](../product/research/PRD-0027/raw-source-preservation.md).

**Next work:** ChatGPT **Desktop** local store — locate and
content-blind inventory only. Do not ingest until an explicit
allowlisted path and RED. No second source (Cursor/Claude) unless
asked. No interpretation of the export corpus without a new RED
envelope.

---

## Completion — V1 readiness review

Return when a candidate can: configure a scope; automatically observe
changes; preserve changed bytes; deduplicate unchanged; honest
provenance/clocks; resolve evidence; STOP; forget-scope; survive
restart; distinguish canonical vs rebuildable; no Activity.

Then report what the operator can do now, limitations, and the exact
proposed real-data pilot — **wait** for authorization before capturing
personal source.

Also return on genuine RED, a product-intent decision that needs the
operator, or a falsified core premise.

Do not return with a chronological dump of every step.
