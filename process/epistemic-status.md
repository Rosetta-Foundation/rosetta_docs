# Epistemic status (Rosetta)

> **Status:** Proposed (2026-08-01).  
> **Peers:** [research-protocol](research-protocol.md),
> [adversarial-review](adversarial-review.md),
> [historical-parallel](historical-parallel.md).

How we **label confidence** so fluent prose cannot masquerade as settled fact.

---

## 1. Claim type codes

| Code  | Type                        | Notes                                                     |
| ----- | --------------------------- | --------------------------------------------------------- |
| **H** | Historical / empirical fact | Falsifiable against the record                            |
| **P** | Pattern / mechanism         | “When X, civilizations tend to Y” — needs parallels       |
| **I** | Interpretive thesis         | Author synthesis; argue, don’t pretend it’s a measurement |
| **O** | Practitioner observation    | Career/org experience (Research **W6**)                   |
| **M** | Memoir / phenomenological   | Personal; usually non-evidence **N1**                     |
| **N** | Normative / value           | Oughts — no empirical verifier                            |
| **D** | Definitional / analytic     | Clarifies terms; watch smuggled empirics                  |

**Load-bearing (LB):** if overstated, the argument of the section fails.

---

## 2. Confidence vocabulary

Use these four labels (and only these) in status tables:

| Confidence                  | Meaning                                                           |
| --------------------------- | ----------------------------------------------------------------- |
| **Established by evidence** | Multiple independent appropriate-weight sources; safe to build on |
| **Working hypothesis**      | Best current understanding; structural + some evidence; revisable |
| **Speculative**             | Plausible; not yet grounded enough to rely on                     |
| **Contested**               | Credible unresolved disagreement — present sides honestly         |

Normative (**N**) claims are **not** scored with this vocabulary as if they were
facts. Mark them `Normative` (or flavor “no empirical verifier”) and argue
values explicitly.

---

## 3. Epistemic status table (template)

End claim-heavy drafts, research passes, and adversarial rounds with:

```markdown
| Claim | Type | Confidence         | Basis | What would change this |
| ----- | ---- | ------------------ | ----- | ---------------------- |
| …     | H    | Working hypothesis | …     | …                      |
```

**Rules:**

- Downgrade as readily as upgrade.
- “What would change this” must name a _falsifier_ or decisive observation, not
  vibes.
- Same-lineage AI self-review produces **upper-bound** confidence only — label
  it as such until a cross-lineage or human pass lands
  ([adversarial-review](adversarial-review.md)).

---

## 4. Instance

Living example: [`../story/CLAIMS-CATALOG.md`](../story/CLAIMS-CATALOG.md)
(priority queue + draft spine table for the early-access essay).
