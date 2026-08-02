# Cross-lineage adversarial prompt (ready to paste)

**Purpose:** Get a **true** adversarial pass on the story’s epistemic status
table from a **different model family** (or a human reviewer). The
same-lineage upper bound lives in [`ADVERSARIAL-REVIEW.md`](ADVERSARIAL-REVIEW.md)
and must **not** be treated as final.

**How to use:** Paste everything below the line into another model (e.g. GPT,
Gemini, Claude-from-another-vendor account, Grok) or give it to a human
historian / SE knowledge-management reviewer. Ask them to return answers in
the same structure. File their response as a new section or PR against this
repo — do not merge it into `ADVERSARIAL-REVIEW.md` without labeling the
reviewer lineage.

**After the pass:** Update the confidence column in
[`ADVERSARIAL-REVIEW.md`](ADVERSARIAL-REVIEW.md) only where the new review
**lowers** confidence or adds a concrete falsifier; never raise confidence from
a single friendly pass.

---

```
You are an adversarial reviewer for an essay series about knowledge,
memory, and software engineering (Rosetta story chapters 1–8). Your job is
to attack overclaiming — not to rewrite the essay or agree with it.

Context (non-negotiable):
- Chapters are essays without inline academic citations.
- Evidence lives in RESEARCH-GROUNDING.md, HISTORICAL-PARALLEL.md, and
  ADVERSARIAL-REVIEW.md (same-lineage only — treat that file as biased upward).
- Normative claims (C8-N*) are value commitments; do not “falsify” them with
  data. Flag only if they are disguised as empirical facts.

For EACH row in the table below, answer:
1. Is the stated confidence too high, too low, or roughly right? Why (2–4
   sentences max)?
2. Strongest counterexample or literature that would force a soften.
3. One concrete “what would change this” test (observable, not vibes).

Then add:
4. The single most dangerous overclaim in chapters 1–8 if a smart skeptic
   read only the essays (no research layer).
5. Any claim missing from the table that should be on it.

Do not invent citations you cannot name. Prefer “I don’t know” over fake
references. Prefer lowering confidence over raising it.

## Epistemic status table (under review)

| Claim | Stated confidence | Basis |
| ----- | ----------------- | ----- |
| C2-H1 Plural independent writing | Established by evidence* | T2 research pack |
| C2-H2 Earliest writing purposes (admin/accounting-heavy) | Working hypothesis (softened) | T2 pack |
| C2-H3 External memory threshold / writing unlock | Working hypothesis (softened) | T2 pack |
| C2-H0 Scale vs memory (oral → cities) | Working hypothesis (directional; no hard cutoff) | T2 pack + parallel |
| C5-O1 Few context-carriers in orgs | Working hypothesis (strong)* | T2 bus-factor pack |
| C5-O2 Loss despite artifacts after departure | Working hypothesis (strong)* | T2 turnover literature |
| C4-H2 / C6-H2 Search ≠ relational understanding | Working hypothesis (strong for known-item gap) | T2 exploratory-search pack |
| C2-P2 / C6-P1 Limit → tool (civilizational pattern) | Working hypothesis (bounded)* | Historical parallel |
| C7-I3 Destinations ≫ paths in ordinary practice | Working hypothesis (comparative)* | Parallel + Ch.7 soft |
| C8-N* Intergenerational obligation | Normative | Value commitment |

*Prior same-lineage review treated these as upper bounds.

Return your review as markdown with one subsection per claim ID, then items 4–5.
```
