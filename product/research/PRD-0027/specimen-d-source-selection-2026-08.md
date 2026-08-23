---
id: PRD-0027-specimen-d-source-selection-2026-08
title: Specimen D — frozen source-selection procedure
date: 2026-08-23
prd: PRD-0027
status: Frozen procedure (corrected before selection)
source: Approved Specimen D design refinements; hasParts join correction 2026-08-23 before any seed
---

# Specimen D — frozen source-selection procedure

Sanitized research procedure for a **new** private specimen. It is not
a continuation of lost E4b / Specimen C artifacts, not a Qwen/local
adapter task, and not authorization to invoke a model.

No conversation, node, title, or source text is recorded here.
Selection has **not** been executed.

Type: **H** (procedure). AI wording is **W7** and is never evidence.

## Identity and store

- Specimen id: **D**
- Durable private root: `~/.local/share/rosetta/specimens/D/`
- Not `/tmp`. Outside git. Deletion is an intentional human action.
- Source export stays where it already is. Do not copy the archive
  into `D/`.
- A later `import-chatgpt` into `D/graphs/` is a **new** graph file,
  not reconstruction of lost derived records.

## Provider (not this step)

- D1, if later approved, uses the existing xAI / `grok-4.6` path.
- Local / Qwen adapter remains separate backlog. Do not implement or
  invoke it for D.

## Frozen selection (replace “first 3–4 text nodes”)

Do not select by title, topic, salience, likely interpretability, or
likely recognition. Do not use the prayer-journal sample. Do not read
source text while applying this procedure.

1. Confirm the existing ChatGPT export still inventories (status and
   structural counts only).
2. Eligible conversations: structural only — at least one eligible
   text node. No title shopping.
3. Draw **one** eligible conversation with a recorded CSPRNG seed.
4. Inside that conversation, build `eligible[]` = text nodes in the
   reconstructed graph’s existing node-array order.
   - Eligible text node: `hasMessage`, `hasParts`, `contentType` is
     exactly `text`, role is `user` or `assistant`.
   - Drop null-message roots and all other content types.
5. Draw **one** index `i` uniformly from `eligible[]` with a recorded
   seed (or the next value from the same recorded CSPRNG stream).
6. Bounded local window, graph/order structure only, radius 1,
   maximum 3 nodes:
   - Center = `eligible[i]`.
   - Add the immediate predecessor and successor in `eligible[]`
     when they exist (`i-1`, `i+1`).
   - Do not skip farther along the list to fill a quota.
   - Do not add ineligible neighbors.
   - A window of 1 or 2 nodes is valid.
7. Record seed, algorithm, conversation id, node ids, and window
   ids **only** under the private `D/` root — not in git.
8. Do not preview statements before a later approved D1.

Same `candidate-observation` policy as E4a: 1–3 observations **or**
insufficient-evidence. Insufficient-evidence is a valid D1.

## Not done in this record

As written at the original freeze (before import, before the
discrepancy below, before any seed):

- No conversation or node was selected.
- No source-graph file was written.
- No model was invoked.
- D1 / D2 / D3 / E8 did not start.

## Correction — 2026-08-23, before any seed

Discovered after `import-chatgpt` wrote the new D graph and **before**
any CSPRNG seed or specimen selection. This is a protocol correction,
not a post-selection adjustment. The original freeze above is left in
place.

**Discrepancy.** The frozen eligible-text-node predicate includes
`hasParts`. The persisted source-graph type (`ChatGptSourceNode`) does
not store `hasParts`; `buildSourceGraph` drops it. `hasParts` exists
only on the in-memory stripped raw node (`ChatGptRawNode`). Selection
was blocked rather than improvising.

**Approved correction (option 3).** Do not change the source-graph
schema. Do not drop `hasParts` from the frozen predicate.

- The persisted source graph supplies conversation identity, node
  identity, node-array order, `hasMessage`, `role`, and `contentType`.
- The original ChatGPT archive / raw reconstruction may be joined **by
  node id only** to recover the structural boolean `hasParts`.
- No raw text, titles, parts contents, topic, salience, or semantic
  information may be inspected during eligibility or selection.
- `hasParts` is used only as a boolean eligibility fact.
- Graph node-array order remains authoritative for `eligible[]`.
- The source archive identity / hash must match the already recorded
  Specimen D corpus identity.

Eligible conversations and `eligible[]` still use the originally frozen
predicate:

`hasMessage && hasParts && contentType === 'text' && role ∈ {user, assistant}`

Conversation list order is the persisted graph’s conversation-array
order, filtered to conversations with at least one eligible node.
