# ADR-0008: Implementation Spec Format — The PRD-to-Build Contract

**Status:** Accepted

**Date:** 2026-07-31

---

> An implementation spec is one Markdown file per PRD phase, living in the
> target repository under `specs/<PRD-ID>/phase-<n>-spec.md`. Structured
> frontmatter carries identity and the blast-radius envelope; the body carries
> tasks with acceptance criteria tagged by verification tier (`test:`,
> `agent:`, `manual:`). Approval is a status flip in a dedicated commit — the
> human gate of PRD-0011, recorded as a reviewable, dated event.

---

# Background

The glossary defines the **implementation spec** as the contract between a PRD
("what and why") and the build (PRD → phased implementation specs → build) —
but no format for it exists. PRD-0011's evidence-gated workflow makes the gap
blocking: its single human gate approves "the spec + envelope," its machine
gates verify "executable acceptance criteria," and its `SpecTask` / `Envelope`
contracts must serialize _somewhere_. Every downstream piece of machinery —
spec generation, verification suites, agent-driven interface verification,
envelope enforcement — consumes this format.

Adopter workspaces are equally blocked: the per-repo org override in
`rosetta_dev-scripts` exists so adopters can run Rosetta machinery against
their own app repos, and the spec format is the contract their agents author
and consume. It must therefore be foundation-neutral (per the CONTRIBUTING
vocabulary rule) and repo-agnostic.

# Decision

## 1. One spec per PRD phase, legible to humans and machines

Each rollout phase of a PRD gets exactly one implementation spec: a Markdown
document with structured YAML frontmatter, following the same
dual-legibility rule as PRDs. Agents author, consume, and act on specs; humans
review and approve them.

## 2. Specs live in the target repository

Specs are committed to the repository the PRD's implementation lands in — not
to `rosetta_docs`:

```
<target-repo>/specs/<PRD-ID>/phase-<n>-spec.md
e.g. rosetta_chronicle/specs/PRD-0011/phase-1-spec.md
```

Specs travel with the code they gate: implementation agents in worktree
isolation read them locally, verification runs against the same checkout, and
adopter repos get the identical layout. The PRD links back via its
`related_specs` frontmatter field (e.g.
`[specs/PRD-0011/phase-1-spec.md]`).

## 3. Frontmatter carries identity and the envelope

```yaml
---
id: SPEC-PRD-0011-P1
prd: PRD-0011
phase: 1
status: Draft # Draft | Approved | Done | Superseded
date: YYYY-MM-DD
owner: <name>
envelope:
  allowedPaths: ["src/**", "specs/PRD-0011/**"]
  forbiddenSurfaces: ["migrations", "auth", "ci-config"]
  maxDiffLines: 1500
  budgetK: 200
---
```

The `envelope` block is the serialized form of PRD-0011's `Envelope` contract,
approved together with the spec and enforced by the machine gate.

## 4. Tasks are body sections; criteria are tagged by verification tier

Each task is a section carrying its story reference, complexity, dependencies,
engineering notes, and acceptance criteria:

```markdown
## Task T-01: <title>

- **Story:** S-01
- **Complexity:** M # S | M | L
- **Depends on:** [] # task IDs

<Engineering notes.>

### Acceptance criteria

- [ ] test: <deterministic condition, asserted by a scripted test in CI>
- [ ] agent: <condition verified by an agent using the running sandbox
      interface, evidence attached to the verdict>
- [ ] manual: <condition only a human can verify — see below>
```

Every criterion **must** carry a tier tag:

| Tag       | Verified by                                                     |
| --------- | --------------------------------------------------------------- |
| `test:`   | Scripted tests in CI — the deterministic regression floor       |
| `agent:`  | Agent-driven interface verification against the running sandbox |
| `manual:` | A human — escape hatch; its presence disables auto-advance      |

A phase containing any `manual:` criterion escalates at its boundary instead
of auto-advancing. This keeps the tag honest: it is a deliberate opt-out of
the evidence gate, not a default.

## 5. Approval is a status flip in a dedicated commit

Mirroring ADR acceptance: flipping `status: Draft` → `Approved` happens in a
small dedicated commit (`docs: approve SPEC-PRD-NNNN-Pn`), making the human
gate a reviewable, dated event in the target repo's history. After approval,
any change to tasks, criteria, or envelope reverts `status` to `Draft` —
re-approval required. `Done` is set when the phase's machine gate passes;
`Superseded` when a later spec replaces it.

## 6. The canonical template lives beside the PRD template

`product/SPEC-TEMPLATE.md` in `rosetta_docs` is the canonical starting point,
kept next to the PRD `TEMPLATE.md` it is sliced from. Target repos copy it;
`rosetta_dev-scripts` may later lay it down automatically (see Adoption).

# Consequences

**Positive:**

- PRD-0011's machinery has a concrete contract: spec generation has an output
  format, machine gates have criteria to execute, envelope enforcement has a
  schema, and `PhaseVerdict` evidence maps one-to-one onto tagged criteria.
- The approval gate is auditable in git history, in the same repo the code
  lands in — provenance travels with the clone (ADR-0005).
- Adopters consume the same format in their own repos with no
  Rosetta-Foundation coupling beyond this public ADR and template.
- The `manual:` escape hatch makes partially-automatable phases expressible
  without weakening the auto-advance rule.

**Negative / costs:**

- Template copies can drift across repos until dev-scripts automates laying
  them down.
- Tag discipline is load-bearing: an untagged criterion is a spec-format error
  and must fail spec validation in the workflow runtime.
- YAML frontmatter as the envelope's source of truth means the workflow must
  parse Markdown frontmatter — acceptable, as every other Rosetta artifact
  already works this way.

# Adoption

1. **rosetta_docs** — this ADR, `product/SPEC-TEMPLATE.md`, and the PRD
   template's `related_specs` example updated to the new path convention.
2. **PRD-0011 workflow (rollout Phase 1)** — spec generation emits this
   format; spec validation rejects untagged criteria.
3. **rosetta_dev-scripts** — later: lay `specs/` scaffolding and the template
   into repos via team-setup, so copies stop drifting.
4. **Adopters** — author specs in their own repos from the public template;
   the CONTRIBUTING vocabulary rule keeps upstream contributions
   foundation-neutral.
