# Research Protocol (Rosetta lite)

> **Status:** Proposed (2026-08-01).  
> **Peer protocols:** [epistemic-status](epistemic-status.md),
> [adversarial-review](adversarial-review.md),
> [historical-parallel](historical-parallel.md).

How we gather and cite evidence for claims in Rosetta writing (story,
foundations, PRDs, ADRs). This is **citation integrity and source weighting**,
not a proof that the world is as we say — that pressure comes from adversarial
review and historical parallel tests.

---

## 1. Choose a scope tier deliberately

| Tier                        | When                                                            | Source target                                | Verification                                                 | Output                                                                                       |
| --------------------------- | --------------------------------------------------------------- | -------------------------------------------- | ------------------------------------------------------------ | -------------------------------------------------------------------------------------------- |
| **T2 — Targeted gap-close** | A named claim or claim cluster is load-bearing and undersourced | 3–7 sources per gap                          | Separate session/model preferred; human spot-check ≥1 source | Research grounding subsection (below) or durable notes under `story/sources/` / topic folder |
| **T3 — Inline grounding**   | Supporting color or low-load claims                             | 3 sources if load-bearing; 1 if illustrative | Authoring agent self-check                                   | Inline citations in the artifact                                                             |

**T1 programmatic sweeps** (dozens of digests + a maintained index) are
_reserved_ until Rosetta has multiple contested doctrine debates. Do not invent
a `SOURCE_INDEX` for a single essay.

**Escalate** T3 → T2 when a claim becomes load-bearing for a public or
decision-facing artifact.

---

## 2. Source weights (what counts for what)

Weights are relative to the _kind of claim_, not a prestige ranking of outlets.

| Weight | Source type                                                                                                             | Best for                                  |
| ------ | ----------------------------------------------------------------------------------------------------------------------- | ----------------------------------------- |
| **W1** | Peer-reviewed journals; university-press monographs; major statistical agencies                                         | Empirical / historical claims             |
| **W2** | Domain peer-review-equivalent (field flagships, NBER/IMF working papers in econ, ACM/IEEE in CS, top law reviews, etc.) | Field-pace literature                     |
| **W3** | Primary documents (statutes, filings, treaties, institutional minutes)                                                  | What was said or done by whom             |
| **W4** | Investigative journalism with corrections culture; transparent reference indices                                        | Contemporary facts                        |
| **W5** | Think tanks, named-expert essays, quality long-form                                                                     | Contested positions — **label viewpoint** |
| **W6** | Identifiable practitioner accounts                                                                                      | Tacit / operational knowledge             |
| **W7** | AI synthesis / prior agent output                                                                                       | **Diagnostic only — never evidence**      |
| **W8** | Unsourced confident assertion                                                                                           | Forbidden for load-bearing claims         |

**History / cognitive science / SE research** are the usual buckets for Rosetta
story and product claims. Balance _actually contested_ sides; do not
false-balance a converged scholarly consensus.

**Historically contested, now converged:** cite the contemporary convergence,
name the older foil, and flag live critiques of the _new_ consensus as future
T2 work — do not resurrect a dead minority for theater.

---

## 3. Non-evidence ladder

Material can be _admissible in the conversation_ without being _evidence for a
claim_. Each rung needs a firewall against silent upgrade:

| Rung                            | What                                      | Firewall                                                                                                 |
| ------------------------------- | ----------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| **N1 — Illustrative**           | Intuition pump; essay survives if swapped | Label illustrative; cannot carry load-bearing claims alone                                               |
| **N2 — Cultural artifact**      | Fiction/film/art as worked example        | Ground plot facts to the work; themes are value-contested, not proof                                     |
| **N3 — Contested or laundered** | Contested W5, or W7 summary of a source   | **De-launder:** read the underlying source; restore contestation markers; never let W7 stand as evidence |

---

## 4. Source-count floors

| Scope                 | Floor | Soft ceiling           |
| --------------------- | ----- | ---------------------- |
| T2 per named gap      | 3     | 7                      |
| T3 load-bearing claim | 3     | 5 (then promote to T2) |
| T3 illustrative claim | 1     | —                      |

**Every source gets a URL** (open access preferred). Books without full text
online: full biblio + publisher/OpenLibrary/JSTOR/WorldCat link + page/chapter
locator.

---

## 5. Citation-integrity checks (§ verification)

Run before merging research into a public-facing draft. Prefer a **different
session or model** than the author for T2.

1. **Existence** — URL resolves; author/venue real.
2. **Tier honesty** — W1 is not a blog post; W5 is not dressed as W1.
3. **Quote fidelity** — excerpts verbatim; locator present.
4. **Balance** — contested claims show credible sides (or named convergence).
5. **No phantoms** — no invented section titles or fake papers.

**This does not prove the conclusion.** A real, well-cited source can still be
wrong or misused. Adversarial review and historical parallel cover that gap.

---

## 6. Link-and-quote discipline

For every citation:

1. Live URL
2. Author, title, venue, date
3. Page/section locator for long works
4. Verbatim quotes with locators
5. Viewpoint/tier label when the claim is contested

---

## 7. T2 output — Research grounding subsection

Prefer this lightweight form until sources must be reused across many artifacts:

```markdown
### Research grounding — [claim id or cluster]

Sources consulted (N):

1. [Author, Title, Venue, Date](URL) — W# [, viewpoint]. Relevance: …
2. …

Aggregate finding: …

Verification: citation-integrity checks passed by [agent/human], [date].
```

Durable digests under `story/sources/` (or a future `sources/`) are optional
when the same sources will be cited repeatedly.

---

## 8. Fit with other Rosetta machinery

- **PRD-0011 / sdlc-workflow** — verifies _ships_, not essay sentences.
- **ADR-0007 / Chronicle** — provenance of activity artifacts.
- **This protocol** — provenance and weight of _citations behind claims_.
- **PRD-0009 coherence protocol** — org-chronicle promotion gate; **not** the
  same as a citation audit. If we add a phantom-citation audit later, name it
  **source audit** / **citation integrity**, not “coherence.”
