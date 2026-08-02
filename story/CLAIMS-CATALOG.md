# Story claims catalog (Chapters 1–8)

Working inventory of **assertive content** in the early-access Rosetta essay,
typed with Rosetta [`process/`](../process/README.md) so we can verify
load-bearing history, strengthen contested points, or soften prose where
reality is thinner than the sentence sounds.

This file is a **research instrument**, not part of the essay voice. Readers of
the story do not need it; writers and reviewers do.

**Source chapters:** [`chapter-1.md`](chapter-1.md) … [`chapter-8.md`](chapter-8.md)  
**Protocols:** [`../process/`](../process/README.md) (canonical). Inspiration:
Civic Blueprint’s research / adversarial / parallel stack (adapted, not forked).

---

## Claim types

See [`../process/epistemic-status.md`](../process/epistemic-status.md) for the
full definitions. Short form:

| Code  | Type                        | Instrument                                               |
| ----- | --------------------------- | -------------------------------------------------------- |
| **H** | Historical / empirical fact | [Research Protocol](../process/research-protocol.md)     |
| **P** | Pattern / mechanism         | [Historical Parallel](../process/historical-parallel.md) |
| **I** | Interpretive thesis         | [Adversarial Review](../process/adversarial-review.md)   |
| **O** | Practitioner observation    | Research **W6**                                          |
| **M** | Memoir / phenomenological   | Non-evidence **N1**                                      |
| **N** | Normative / value           | No empirical verifier                                    |
| **D** | Definitional / analytic     | Low research load                                        |

**Confidence vocabulary:** Established by evidence · Working hypothesis ·
Speculative · Contested ([epistemic-status](../process/epistemic-status.md)).

**Load-bearing (LB):** if false or overstated, the chapter’s argument wobbles.

---

## Workflow (this essay)

1. Tag assertive sentences here.
2. For each **LB** claim: T2 or T3 + source floor
   ([research-protocol](../process/research-protocol.md)).
3. Write a Research grounding subsection (W1–W4 + URLs).
4. Citation-integrity check in a **different** session/model.
5. **P**-claims: Historical Parallel (supporting **and** challenging cases).
6. Epistemic status table; soften prose to match.
7. Keep **N**-claims as oughts — do not “verify” into facts.
8. Early-access challenges:
   [challenge-claim.yml](../.github/ISSUE_TEMPLATE/challenge-claim.yml).

---

## Priority verification queue

Ordered by (load × falsifiability × how much the prose currently asserts).

| Pri | ID            | Claim (short)                                                                   | Suggested process pass                             | Provisional status                                                                                                                       |
| --- | ------------- | ------------------------------------------------------------------------------- | -------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | C2-H1         | Writing appeared independently in several parts of the world                    | T2 Research + Historical Parallel                  | **Likely established** (mainstream: ≥3–4 pristine inventions; Egypt↔Mesopotamia stimulus still debated)                                  |
| 2   | C2-H2         | Earliest writing was primarily practical (harvests, trade, law, agreements)     | T2 (W1/W2 history monographs)                      | **Partially corroborated** — true for early cuneiform administrative corpus; not universal (e.g. Chinese oracle bones)                   |
| 3   | C2-H3         | Writing first allowed knowledge to exist independently of living carriers       | T3 + adversarial on “first”                        | **Working hypothesis** — proto-writing / tokens / monumental art complicate “first”; keep as _qualitative threshold_, not absolute first |
| 4   | C1-H1 / C2-H0 | Small communities could rely on memory; scale broke that balance                | T2 anthropology / cognitive history                | **Working hypothesis** — directionally standard; avoid implying a sharp population cutoff                                                |
| 5   | C5-O1→P1      | Orgs depend on few context-carriers; departure destroys continuity despite docs | T2 SE literature (bus factor, knowledge loss) + W6 | **Working hypothesis** with **empirical cousins** (bus/truck factor; concept-keeper loss)                                                |
| 6   | C4-H1 / C6-H1 | Search retrieves documents but not relational understanding                     | T2 HCI / IR / cognitive science                    | **Working hypothesis** — strong as _limitation of keyword retrieval_; weaken absolute “cannot”                                           |
| 7   | C6-P1 / C2-P1 | Each coordination limit → new preservation/navigation tool                      | Historical Parallel                                | **Working hypothesis** — thesis spine; needs challenging cases (dark ages, lost libraries, oral cultures that scaled)                    |
| 8   | C7-I1         | Civilization is better at destinations than paths                               | Adversarial + Historical Parallel                  | **Working hypothesis / interpretive**                                                                                                    |
| 9   | C8-N\*        | Intergenerational obligation to leave traversable paths                         | No empirical verifier                              | **Normative** — keep; do not fact-check into science                                                                                     |

---

## Catalog by chapter

Legend: **LB** = load-bearing · **Soft?** = candidate wording change if evidence is thin.

### Chapter 1 — Before Civilization

| ID    | Quote / paraphrase                                                                             | Type | LB  | Notes / action                                                                                        |
| ----- | ---------------------------------------------------------------------------------------------- | ---- | --- | ----------------------------------------------------------------------------------------------------- |
| C1-I1 | Tech ages (stone→information) are real but may be _evidence_ of deeper progress, not the cause | I    | ✓   | Thesis setup; adversarial: steelman tech-determinist counter                                          |
| C1-H1 | Early humans lacked speed, armor, claws relative to many animals                               | H    |     | Soft comparative biology; one W1/W2 source enough; or soften to “few obvious physical advantages”     |
| C1-H2 | Humans accumulated knowledge and transmitted it across generations                             | H    | ✓   | Established at high level (cultural transmission literature)                                          |
| C1-I2 | Civilization may begin in ordinary teaching moments                                            | I/N  | ✓   | Interpretive origin story — label as “may”; not a dating claim                                        |
| C1-H3 | Shared imperfect knowledge outperforms isolated perfect knowledge                              | H/P  | ✓   | Aligns with cumulative culture research; T2 cite (e.g. cultural evolution)                            |
| C1-H4 | For most of history knowledge died with carriers (healers, elders)                             | H    | ✓   | Directionally true; avoid implying _no_ external memory before writing (songlines, monuments, tokens) |
| C1-I3 | Bottleneck was preservation, not intelligence                                                  | I    | ✓   | Thesis; adversarial + challenging cases (censorship, inequality of access)                            |
| C1-I4 | Defining inventions enable collective memory more than individual power                        | I    | ✓   | Spine claim through Ch. 8                                                                             |

### Chapter 2 — When Memory Was No Longer Enough

| ID     | Quote / paraphrase                                                                           | Type | LB  | Notes / action                                                                                                                                                                                                                                      |
| ------ | -------------------------------------------------------------------------------------------- | ---- | --- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| C2-H0  | Small campfire communities: memory “sufficient”; little needed for survival                  | H    | ✓   | Soft?: “often sufficient” / avoid Dunbar-as-fact. Challenging: large oral polities                                                                                                                                                                  |
| C2-H0b | Settlement, surplus, cities, trade, law created memory load beyond individuals               | H    | ✓   | Standard agricultural/urbanization narrative — T3 cite                                                                                                                                                                                              |
| C2-P1  | Success of cooperation created demands cooperation alone couldn’t satisfy                    | P    | ✓   | Historical Parallel                                                                                                                                                                                                                                 |
| C2-H1  | **“Writing appeared independently in several parts of the world.”**                          | H    | ✓✓  | **User spark.** Mainstream: Mesopotamia, Egypt, China, Mesoamerica as (at least) independent; Egypt–Mesopotamia _stimulus diffusion_ still debated. Soft?: “in several parts of the world, and at least some of those inventions were independent.” |
| C2-H2  | Earliest purpose of writing was practical (harvests, trade, laws, agreements)                | H    | ✓   | Soft for universality — administrative Near East yes; Shang oracle bones ritual/divinatory                                                                                                                                                          |
| C2-H3  | Writing let knowledge exist independently of possessors                                      | H/I  | ✓   | Keep as threshold claim; note precursors (tokens, seals) if pedantic reviewers                                                                                                                                                                      |
| C2-H4  | Paper, libraries, print, universities/archives/scientific societies extended external memory | H    |     | Broadly established; T3 color citations if desired                                                                                                                                                                                                  |
| C2-I1  | Preservation aims to let each generation start where the last ended                          | I/N  | ✓   | Near-normative; fits thesis                                                                                                                                                                                                                         |
| C2-P2  | Each coordination limit → tools that extend the limit (share → write → …)                    | P    | ✓   | Core progressive pattern; needs challenging cases                                                                                                                                                                                                   |

### Chapter 3 — Growing Up Between Eras

| ID    | Quote / paraphrase                                                                          | Type | LB  | Notes / action                                                            |
| ----- | ------------------------------------------------------------------------------------------- | ---- | --- | ------------------------------------------------------------------------- |
| C3-I1 | People in transitions rarely recognize them as historic                                     | I    |     | Soft social claim; optional T3                                            |
| C3-H1 | Late 20th c.: PCs, phone lines carrying data, networks opening to families                  | H    |     | Established popular history; memoir frame OK                              |
| C3-M1 | Grandmother/computer-lab anecdote                                                           | M    |     | N1 — verify only if published as fact about named living people (privacy) |
| C3-M2 | AOL / ~16.6 kbps dial-up details                                                            | M/H  |     | Harmless if approximate; “on the order of” is safer                       |
| C3-O1 | Tech changes faster than the problems it solves                                             | O/I  | ✓   | Practitioner aphorism; adversarial: some problem classes _do_ get solved  |
| C3-H2 | Lifetime shift: information access from scarce → ordinary (search, wiki, OSS, open courses) | H    | ✓   | Established at civilizational scale; hedge “anyone” (digital divides)     |
| C3-I2 | Preserving + accessing got easier — incomplete feeling remains                              | I    | ✓   | Bridge to Ch. 4; not a fact claim                                         |

### Chapter 4 — When Information Became Ordinary

| ID    | Quote / paraphrase                                                                             | Type | LB  | Notes / action                                          |
| ----- | ---------------------------------------------------------------------------------------------- | ---- | --- | ------------------------------------------------------- |
| C4-O1 | Early career: solving problems was largely _finding_ information                               | O    | ✓   | W6; widely attested in SE folklore + empirical SE       |
| C4-H1 | Search/docs/communities collapsed time-to-find                                                 | H/O  | ✓   | Established directionally                               |
| C4-O2 | New difficulty: fragmentation; hard to see how pieces fit                                      | O    | ✓   | LB for Rosetta thesis                                   |
| C4-D1 | Information ≠ understanding; understanding is relational                                       | D/I  | ✓   | Analytic spine; optional cognitive-science T3           |
| C4-H2 | Search retrieves docs but not disagreement structure, assumptions, historical meaning          | H    | ✓   | Soft “cannot easily” (already soft); cite IR/HCI limits |
| C4-H3 | Humans learn by connecting (child language, science, history, engineering, associative memory) | H    |     | High-level; one cognitive-science survey suffices       |
| C4-P1 | Solving preservation/distribution reveals subtler limits                                       | P    | ✓   | Same pattern family as C2-P2                            |
| C4-I1 | Generation’s question: abundance → understanding                                               | I    | ✓   | Thesis                                                  |

### Chapter 5 — The Fragility of Context

| ID    | Quote / paraphrase                                                                     | Type | LB  | Notes / action                                                   |
| ----- | -------------------------------------------------------------------------------------- | ---- | --- | ---------------------------------------------------------------- |
| C5-O1 | Successful orgs rely on a few people who hold relational context                       | O    | ✓✓  | Maps to bus factor / “concept keepers”; T2 cite SE studies       |
| C5-O2 | Docs/code remain after departure but capability drops                                  | O/H  | ✓✓  | Empirical support in turnover / knowledge-loss literature        |
| C5-O3 | Pattern not unique to SE (labs, families, communities)                                 | O/P  | ✓   | Historical Parallel / anthropology; don’t overclaim universality |
| C5-D1 | Context = assumptions, alternatives, conversations, influences — not mere files        | D    | ✓   | Definitional for Rosetta                                         |
| C5-I1 | Continuity across generations depends on relationships harder to preserve than records | I    | ✓   | Spine                                                            |
| C5-I2 | Civilization inherits questions/mistakes/stories held by context                       | I    | ✓   |                                                                  |

### Chapter 6 — Navigating Knowledge

| ID    | Quote / paraphrase                                                                      | Type | LB  | Notes / action                                                      |
| ----- | --------------------------------------------------------------------------------------- | ---- | --- | ------------------------------------------------------------------- |
| C6-P1 | Major knowledge advances improve navigation as well as preservation                     | P    | ✓   | Parallel test: writing, libraries, print indexes, hypertext, search |
| C6-H1 | TOC, catalogs, citations, hyperlinks are underrated navigation tech                     | H/I  |     | Soft; historically defensible                                       |
| C6-H2 | Search works when you know the query; weak for unknown-unknowns / cross-vocab analogies | H    | ✓   | Strong IR/HCI claim — T2                                            |
| C6-O1 | Most valuable people navigate relationships, not just store facts                       | O    | ✓   | Ties to C5                                                          |
| C6-H3 | Tools point to documents; humans perceive meaning relations                             | H/I  | ✓   | Soft absolute; “largely”                                            |
| C6-O2 | Newer AI systems suggest connections / explore unframed questions; imperfect            | O/H  | ✓   | Fast-moving; date the claim; avoid product pitch                    |
| C6-P2 | Navigation methods upgrade when inherited knowledge outgrows prior means                | P    | ✓   |                                                                     |
| C6-I1 | Next challenge: preserve journeys/paths, not only navigate better                       | I    | ✓   | Bridge to Ch. 7                                                     |

### Chapter 7 — The Paths Between Ideas

| ID    | Quote / paraphrase                                                              | Type | LB  | Notes / action                                                                                                                     |
| ----- | ------------------------------------------------------------------------------- | ---- | --- | ---------------------------------------------------------------------------------------------------------------------------------- |
| C7-D1 | Formal paths: TOC, citation, hyperlink, index                                   | D    |     |                                                                                                                                    |
| C7-O1 | Important paths live in conversation / remembered decision sequences            | O    | ✓   |                                                                                                                                    |
| C7-O2 | Inherited systems: destination documented, evolution/reasoning faint            | O    | ✓   | Classic SE “why” gap                                                                                                               |
| C7-I1 | More docs ≠ preserved paths; volume can obscure trails                          | I    | ✓   | Challenging case: excellent design-history cultures                                                                                |
| C7-H1 | Understanding depends on sequence as well as substance (science, law, software) | H/I  | ✓   |                                                                                                                                    |
| C7-I2 | Inheritance of conclusions alone is incomplete                                  | I/N  | ✓   |                                                                                                                                    |
| C7-I3 | Better at accumulating destinations than preserving routes                      | I    | ✓✓  | Adversarial + parallel (oral tradition, Talmudic commentary, git history, ADR culture as counterexamples that _do_ preserve paths) |

### Chapter 8 — What Every Generation Leaves Behind

| ID    | Quote / paraphrase                                                            | Type | LB  | Notes / action                                                    |
| ----- | ----------------------------------------------------------------------------- | ---- | --- | ----------------------------------------------------------------- |
| C8-H1 | Each generation inherits unfinished world (language, cities, laws, tools)     | H    |     | Near-truism                                                       |
| C8-N1 | Obligation to leave paths traversable, not only to discover                   | N    | ✓   | Flavor-(c); keep as ought                                         |
| C8-I1 | Solving problems while erasing reasoning can impoverish successors            | I    | ✓   |                                                                   |
| C8-N2 | Local acts (parent, teacher, engineer) are where continuity is kept or broken | N/O  | ✓   |                                                                   |
| C8-P1 | Faster knowledge production → more need for deliberate intelligibility        | P    | ✓   | Parallel: print explosion → indexes; web → search; ? → path tools |
| C8-I2 | Durable contributions often look modest (writing, libraries, indexes)         | I    |     | Historical color — T3 optional                                    |
| C8-N3 | Selective preservation; forgetting can be merciful                            | N    |     | Important hedge — keep                                            |
| C8-N4 | Responsibility remains human; tools help or hinder                            | N    | ✓   | Anti-tech-savior; fits WRITING-CONTEXT                            |

---

## Worked example: C2-H1 (independent invention of writing)

**Essay sentence:** “It is perhaps not surprising, then, that writing appeared independently in several parts of the world.”

| Check              | Result                                                                                                                                                                                                                 |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Type               | **H** (historical), LB                                                                                                                                                                                                 |
| CB tier            | **T2** gap-close (3–7 W1/W2 sources)                                                                                                                                                                                   |
| Mainstream framing | Multiple _pristine_ inventions commonly listed: Mesopotamia, Egypt, China, Mesoamerica ([overview](https://en.wikipedia.org/wiki/History_of_writing); scholarly treatments e.g. Boltz; Schmandt-Besserat on Near East) |
| Live contestation  | Whether Egyptian writing is fully independent vs. idea-stimulus from Mesopotamia; counts of “true writing” vs. proto-writing                                                                                           |
| CB balance rule    | Name convergence on _plural_ invention; note Egypt stimulus debate honestly (Research Protocol §2.3 historically-contested-but-converged + live critique)                                                              |
| Falsifier          | Strong evidence that all non-Mesopotamian systems derive by diffusion from one origin                                                                                                                                  |
| Prose options      | (a) keep — “independently…several parts” is mainstream-ok; (b) precision pass — “writing systems arose in several regions, at least some independently”                                                                |

**Do not** cite a W7 chat summary as evidence. Digests → quote W1/W2, then verify URLs/quotes (§4).

---

## Suggested story edits (only where reality is sharper)

These are **candidates**, not mandated rewrites:

1. **C2-H1** — optional precision on independence / Egypt debate.
2. **C2-H2** — “often” or “in many of the earliest corpora” rather than universal “earliest purpose.”
3. **C2-H3** — “for the first time at civilizational scale” or “more reliably than before.”
4. **C2-H0** — “often sufficient” for small communities; avoid implying oral cultures cannot scale.
5. **C3-H2** — acknowledge remaining access inequalities when saying “ordinary.”
6. **C4-H2 / C6-H2** — keep “cannot easily”; avoid “cannot.”
7. **C7-I3** — acknowledge counter-traditions that _do_ preserve paths (commentary chains, lab notebooks, ADRs) so the claim is comparative scarcity, not absolute absence.
8. **C6-O2** — time-stamp AI navigation claims; they age fast.

Memoir and normative passages (most of Ch. 3 personal arc; Ch. 8 oughts) should stay; CB would mark them N1/N, not force footnotes.

---

## Epistemic status table (spine — draft)

Fill “Basis” during the T2 pass; this is the Adversarial Review closing artifact.

| Claim                                                  | Confidence (draft)         | Basis (to complete)                           | What would change this                                                            |
| ------------------------------------------------------ | -------------------------- | --------------------------------------------- | --------------------------------------------------------------------------------- |
| C2-H1 Independent (plural) invention of writing        | Established by evidence*   | *pending T2 cite pack; *caveat Egypt          | Monogenesis revival with strong archaeological consensus                          |
| C2-H2 Early writing primarily administrative/practical | Partially corroborated     | Near East strong; China weaker fit            | Show earliest Chinese/Mesoamerican corpora are non-administrative                 |
| C5-O1/O2 Context-carriers / post-departure loss        | Working hypothesis         | Bus-factor & knowledge-loss SE literature; W6 | Evidence that docs alone preserve “why” at scale                                  |
| C4-D1 / C6-H2 Info ≠ understanding; search ≠ relations | Working hypothesis         | IR/HCI + cognitive science                    | Systems that reliably expose disagreement & assumption structure                  |
| C2-P2 / C6-P2 Coordination-limit → tool pattern        | Working hypothesis         | Historical narrative fit                      | Clear counterexamples where limits didn’t produce tools, or tools preceded limits |
| C7-I3 Destinations ≫ paths                             | Working hypothesis         | Interpretive + SE anecdotes                   | Measure path-preservation traditions that scale                                   |
| C8-N1 Obligation to leave traversable paths            | Normative (not evidential) | Value commitment                              | N/A — revise values, not “disprove”                                               |

---

## Next actions

1. Run **T2 Research grounding** for Priority 1–5 (start with C2-H1…H3 and C5-O1).
2. Add `story/sources/` digests **or** a single `story/RESEARCH-GROUNDING.md` subsection (CB T2 inline form) — prefer the lighter form until load increases.
3. One **adversarial** pass (different model family) on the spine table.
4. Apply only the soft? wording changes that the evidence forces.
5. Optional: publish a “challenge a claim” note for early-access readers using CB’s issue fields.

---

## References for this catalog (meta)

- Rosetta process — [`../process/README.md`](../process/README.md)
- Writing / independent invention overviews — [History of writing](https://en.wikipedia.org/wiki/History_of_writing); Denise Schmandt-Besserat, [The Evolution of Writing](https://sites.utexas.edu/dsb/tokens/the-evolution-of-writing/); William G. Boltz on Chinese writing’s independent status
- Organizational knowledge concentration — bus/truck factor literature (e.g. studies summarized at [Bus factor](https://en.wikipedia.org/wiki/Bus_factor); multimodal bus-factor / concept-keeper research in SE)
- Upstream inspiration (not a dependency) — Civic Blueprint `project-2028/agent/process/` research, adversarial-review, and historical-parallel protocols
