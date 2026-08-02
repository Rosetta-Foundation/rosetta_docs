# Adversarial Review Protocol (Rosetta lite)

> **Status:** Proposed (2026-08-01).  
> **Peers:** [research-protocol](research-protocol.md),
> [epistemic-status](epistemic-status.md),
> [historical-parallel](historical-parallel.md).

How we pressure-test claim sets so multi-agent agreement is not mistaken for
truth. Complements SDLC’s rule that the **implementation agent ≠ verifier
agent** — here the **authoring lineage ≠ adversarial lineage**.

---

## 1. When to run

- A load-bearing claim cluster is about to harden in story, foundations, or a
  decision-facing PRD/ADR section.
- Research grounding (T2/T3) just landed and confidence might be inflated.
- An early-access reader or issue challenges a directional claim.

Skip for pure memoir (**M**) and openly normative (**N**) passages unless they
smuggle empirical content.

---

## 2. Designated adversarial role

At least one pass whose job is to **find flaws**, not refine the draft.

**Prompt skeleton:**

> You are adversarially reviewing Rosetta claims. Do not extend or polish.
> Identify the strongest claims; steelman the best counterarguments. Name
> claims accepted without sufficient evidence. Challenge the central thesis.
> Note missing perspectives whose inclusion would change conclusions.
> Agreement is not the goal.

---

## 3. Different inputs (break convergence)

Combine when possible:

| Option                      | Move                                                                 |
| --------------------------- | -------------------------------------------------------------------- |
| **A — Reduced context**     | Give claims + relevant foundations only — not the full drafting chat |
| **B — Alternative framing** | Present conclusions as assertions to test                            |
| **C — Domain lens**         | e.g. ancient historian; SE knowledge-management researcher           |
| **D — Independent lineage** | **Default:** different model family than the author                  |

**Default discipline:** do **not** run the adversarial round in the authoring
lineage. Same-lineage “adversarial” passes are a labeled fallback
(_pre-mortem / upper-bound only_).

This mirrors PRD-0011’s independent verifier agent — applied to prose claims.

---

## 4. Required closing artifact

Produce an [epistemic status table](epistemic-status.md) for every claim under
review. Prefer downgrades where evidence is thin; propose prose softens rather
than silent footnotes that imply certainty.

---

## 5. Standing questions (all reviewers)

1. **Practitioner feasibility** — would this survive contact with real orgs/teams?
2. **Audience portability** — can a careful non-insider follow it?
3. **Missing perspectives** — whose absence would change the conclusion?
4. **Misuse potential** — how could the framing be co-opted?
5. **Steelman integrity** — are we attacking the best form of the claim?
6. **Falsifiability under revision** — did “fixes” immunize the claim against
   all counterexamples?

---

## 6. Human challenge path

External humans can file
[Challenge a claim](../.github/ISSUE_TEMPLATE/challenge-claim.yml) against
story or doctrine sentences. Treat serious challenges as triggers for this
protocol + research re-grounding.
