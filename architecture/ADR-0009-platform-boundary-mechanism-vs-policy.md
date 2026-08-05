# ADR-0009: Platform Boundary — Rosetta Owns Mechanism, Consumers Own Policy

**Status:** Accepted

**Date:** 2026-08-05

---

> Rosetta owns **mechanism**: the SDLC engine, gate evaluation, wake transport,
> the spec format, and the _schemas_ of the `.sdlc/` contracts. A consumer
> workspace owns **policy**: the _contents_ of those contracts, its surface
> labels, its deploy targets, its domain guardrails. Policy reaches the engine
> only through a declared contract seam — never as an engine code path keyed on
> a consumer's name, path, hostname, or domain rule.

---

# Background

The automated SDLC engine lives in
[`rosetta_dev-scripts`](https://github.com/Rosetta-Foundation/rosetta_dev-scripts)
and must work for any Rosetta-workspace consumer. Its first and (so far) only
production consumer is Comita Health, a healthcare company with PHI obligations
that no other consumer shares.

That is the exact condition under which a generic platform quietly stops being
generic. The pressure is never a decision to specialize; it is a series of small
conveniences — an example in a prompt, a label in a default, a path in an
installer — each individually defensible and collectively fatal to the second
consumer.

The boundary rule already existed and was already being followed: engine PRDs
land in `rosetta_docs`, Comita PRDs in `comita_docs`, and the work items in the
gap analysis were tagged `[upstream]` / `[consumer]` / `[both]` one by one. But
the rule itself was written down in exactly one place —
`comita_docs/prompts/automated-sdlc-gap-analysis.md` §4 — a **consumer scratch
file**. The generic platform's own boundary rule was a guest in the opinionated
workspace it exists to keep at arm's length, invisible to anyone reading
`rosetta_docs` and unenforceable in review of an upstream PR.

This ADR is that rule, in the repository the rule governs.

# Decision

## 1. The split is mechanism versus policy, not generic versus specific

"Keep it generic" is not actionable — every specific thing can be described
generically if you try hard enough. The operative question is which of the two
each piece is:

- A **mechanism** answers _how_ something is evaluated, transported, or
  enforced. It is complete without knowing the domain.
- A **policy** answers _what_ the domain's answer is. It is meaningless without
  the domain.

Fail-closed surface validation is a mechanism; _which_ surfaces are forbidden is
policy. A content-level health probe is a mechanism; _what URL to probe_ is
policy. A reviewer checklist hook is a mechanism; _"never log a patient
identifier"_ is policy.

## 2. Ownership follows that split

| Concern                    | Upstream (Rosetta-Foundation)                | Consumer (e.g. Comita-Health)                |
| -------------------------- | -------------------------------------------- | -------------------------------------------- |
| SDLC engine, gates, wakes  | ✅ owns                                      | —                                            |
| Spec format (ADR-0008)     | ✅ owns                                      | —                                            |
| `.sdlc/*.json` **schemas** | ✅ owns                                      | —                                            |
| `.sdlc/*` **contents**     | —                                            | ✅ owns                                      |
| Surface label _resolution_ | ✅ owns (fail closed on unresolvable)        | —                                            |
| Surface label _vocabulary_ | —                                            | ✅ owns (`payments-phi-boundary`, `auth`, …) |
| Deploy/health _protocol_   | ✅ owns (`SDLC_SANDBOX_SHA`, exit semantics) | —                                            |
| Deploy/health _scripts_    | —                                            | ✅ owns                                      |
| Reviewer prompt structure  | ✅ owns                                      | —                                            |
| Domain review rules        | —                                            | ✅ owns (`.sdlc/review-checklist.md`)        |
| Skill / command templates  | ✅ owns (`team-setup`)                       | ✅ owns which are installed and their config |
| Engine PRDs and ADRs       | ✅ `rosetta_docs`                            | —                                            |
| Product PRDs               | —                                            | ✅ `comita_docs`                             |
| Domain guardrails          | —                                            | ✅ owns (healthcare policy, PHI scope)       |

Engine work lands in `Rosetta-Foundation/rosetta_dev-scripts` first — that is
where the issue tracker lives — and syncs to the `Comita-Health` fork. The
engine never contains a Comita path, label, hostname, or healthcare rule.

## 3. Policy enters through a contract seam or not at all

Every legitimate route for consumer policy to influence engine behaviour is a
declared, schema'd file in the consumer's repo:

| Seam                        | Carries                                                    |
| --------------------------- | ---------------------------------------------------------- |
| `.sdlc/surfaces.json`       | The consumer's surface-label vocabulary and path mapping   |
| `.sdlc/verification.json`   | The commands that constitute "verified" for this repo      |
| `.sdlc/environments.json`   | Sandbox targets, deploy and health commands                |
| `.sdlc/review-checklist.md` | Domain review rules injected into the reviewer prompt      |
| Workspace config            | Which repos are watched, which docs ground planning skills |

If a piece of consumer policy has no seam, the answer is **add the seam
upstream**, not add the policy upstream. A seam is a generic mechanism even when
every consumer fills it differently; that is the entire point.

## 4. Three tests decide any specific question

Apply in order. Any failure means the code is on the wrong side of the line.

1. **The name test.** Does the engine mention a consumer's name, repo, path,
   hostname, or domain concept in a way that _changes behaviour_? If yes, it is
   policy in the wrong repo.
2. **The deletion test.** Delete every consumer repo from the machine. Does the
   engine still build, test, and describe itself coherently? Anything that
   breaks was a dependency, not a default.
3. **The second-consumer test.** Would a non-Comita consumer have to _edit
   engine source_ to adopt this? If yes, the thing they would edit belongs in a
   contract they own.

## 5. Provenance in comments is allowed; keyed behaviour is not

Engine comments cite the run that taught the lesson — "Comita Phase 0b: gh
merge SHA is remote-only" sits above the code it explains. That is provenance,
and it is worth keeping: a reader who wants the evidence can find the run.

The test in §4.1 is deliberately about behaviour. A comment naming a consumer is
history. A `if (repo === 'comita_admissions')` is a fork in the platform.

## 6. Genericity is proven, not asserted

The canary acceptance test must pass against a **non-Comita Rosetta repo** as
well as against `comita_admissions`. Until it has, "the engine is generic" is a
claim about intent. The `.sdlc/` contracts make this cheap: a second consumer is
five files, not a code change — and if it turns out not to be, that is the
finding.

## 7. Known boundary violations at the time of acceptance

Recorded here rather than quietly fixed, because an ADR that pretends the
codebase already complies teaches nothing:

- **The reviewer prompt carries a domain example.** `reviewer-prompt.ts` cites
  "authz, PHI/PII, idempotency, failure modes" as examples of non-obvious
  invariants worth documenting. PHI is a healthcare concept in an upstream
  default. The seam already exists — `.sdlc/review-checklist.md` — so the fix is
  to source domain examples from the consumer's checklist and leave the upstream
  prompt with domain-neutral ones.
- **The continuity daemon installs under a single fixed launchd label.**
  `com.rosetta.sdlc-daemon` is a machine-global constant, so installing from a
  second workspace evicts the first. The paths are derived from the installer's
  own location (an earlier hardcoded sibling-workspace path is gone), but the
  label must be workspace-qualified before two workspaces can run at once. The
  daemon scripts also live in the consumer workspace rather than as an upstream
  template, which is why the label was never parameterized.

Neither is load-bearing today (one consumer, one daemon), which is precisely how
both survived.

# Consequences

**Positive:**

- The boundary is reviewable. "This belongs in the consumer's contract" is now a
  citation, not an opinion, in an upstream PR.
- The three tests in §4 are mechanical enough to apply to a diff without a
  judgement call about intent.
- Naming the current violations makes them finite. Two items, both small, both
  now written down where the next reader will find them.

**Negative / costs:**

- Adding a seam is strictly more work than adding the policy inline, and the
  cost lands on whoever hits the case first while the benefit lands on the
  second consumer who may not exist yet. This ADR makes that trade binding
  rather than optional, and it will feel like overhead in the moment.
- Some legitimately shared judgement — what "good docs" means, what a sensible
  diff budget is — has no clean home. It sits upstream as a default and gets
  overridden by contract, which means upstream defaults will occasionally read
  as opinions.

# Adoption

1. **`rosetta_docs`** — this ADR, added to the architecture Records table.
2. **`comita_docs`** — `prompts/automated-sdlc-gap-analysis.md` §4 points here as
   canonical instead of holding the rule itself.
3. **`rosetta_dev-scripts`** — replace the PHI/PII example in the reviewer prompt
   with domain-neutral examples; domain rules arrive via
   `.sdlc/review-checklist.md`.
4. **Continuity daemon** — qualify the launchd label per workspace before a
   second workspace runs one, and promote the daemon scripts to a `team-setup`
   template.
5. **Canary** — run the acceptance test against one non-Comita Rosetta repo.
