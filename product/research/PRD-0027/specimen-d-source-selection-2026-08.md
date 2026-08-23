---
id: PRD-0027-specimen-d-source-selection-2026-08
title: Specimen D — frozen source-selection procedure
date: 2026-08-23
prd: PRD-0027
status: Frozen procedure
source: Approved Specimen D design refinements; selection not executed
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

- No conversation or node was selected.
- No source-graph file was written.
- No model was invoked.
- D1 / D2 / D3 / E8 did not start.
