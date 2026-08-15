---
id: PRD-0024
title: Planning-Side Role Skills
status: Accepted # Draft | Proposed | Accepted | Shipped | Superseded | Deprecated
date: 2026-08-04
owner: Russ Watson
related_adrs: [ADR-0008]
related_specs: []
supersedes: null
---

# PRD-0024: Planning-Side Role Skills

> Strategist, product-manager, and UX-designer roles as workspace skills for
> the collaborative planning session — grounding docs declared by the
> consumer workspace, outputs shaped so the downstream machine can consume
> them — deliberately human-in-the-loop, never autonomous.

## 1. Overview & Goals

### 1.1 Purpose

The front half of the SDLC (intake assessment, strategy, UX design) exists
only as prose in the process description. There is no strategist prompt
anywhere in the workspace; business objectives live half-written in a
website-brief-shaped workbook; UX has method docs and tokens but no role,
no artifact convention, and no spec section for UX acceptance criteria — so
verifier agents have nothing to check states/empty/error/responsive against.
These are front-loaded human judgment stages by design ("the prepped
campfire"), so the right machinery is skills that make the planning
conversation rigorous and its outputs machine-consumable — not autonomous
agents. This PRD ships the three role skills as generic templates and the
spec/template hooks their outputs flow into.

### 1.2 Goals

- `/strategize` assesses an intake idea against the workspace's declared
  business-objective docs and returns a structured recommendation
  (proceed-with-shape / improve / park / decline, with reasons and risks) —
  and flags explicitly when the objectives it needs are not written down,
  rather than inventing them.
- `/design-ux` produces the design artifact the implementation team builds
  from (fidelity proportional to risk) anchored on the workspace's declared
  design method and tokens, and emits UX acceptance criteria (states,
  empty/error states, responsive behavior) in the tiered criteria format
  the verification gate already consumes.
- The PM path (`/write-prd`, `/write-bug-spec`) gains a right-sizing
  checkpoint: story count and scope sanity against PRD size, guarding the
  known agent over-decomposition failure before decompose runs.
- All three skills are generic templates parameterized by a
  workspace-declared grounding config; no consumer document path, product
  name, or domain rule appears in skill bodies.
- The spec template carries a UX-criteria section so UX requirements
  survive from planning into the machine gates without translation.

### 1.3 Non-Goals

- No autonomous strategy or design agents: these skills run inside the
  human planning conversation and their outputs are inputs to human
  judgment — the spec + envelope Approve remains the last human word.
- No design-tool integration (Figma, Storybook pipelines) in this PRD;
  artifacts are markdown/asset files in the docs repo.
- Does not write the consumer's business objectives or design tokens —
  it consumes what the workspace declares and flags gaps.
- Does not add machinery gates before decompose; a skipped `/strategize`
  is a process choice, not an enforced block.

### 1.4 Acceptance Criteria

- [ ] Running `/strategize` on a sample intake in a configured workspace
      produces a structured assessment citing the declared objective docs,
      with an explicit recommendation and at least risks + missing-context
      sections; in a workspace with no objectives declared, it flags the
      gap as its first finding instead of fabricating objectives.
- [ ] Running `/design-ux` on an approved strategy produces a design
      artifact file (sketch/wireframe/comp reference) plus UX acceptance
      criteria rendered in the tiered `test:` / `agent:` / `manual:` format
      that `sdlc-workflow` verification consumes unchanged.
- [ ] The spec template (and bug-spec template where applicable) includes a
      UX acceptance criteria section, and `prd-lint` / spec validation
      accepts specs carrying it.
- [ ] `/write-prd` surfaces a right-sizing check (stories vs PRD scope)
      before hand-off to decompose, and its output still passes `prd-lint`
      on the first try.
- [ ] Skill bodies contain zero consumer-specific paths or names; a
      consumer workspace configures grounding docs in its own config file,
      and the first consumer workspace config points at its workbook, foundations,
      and design-method docs as the first consumer.
- [ ] The skills are distributed through the existing team-setup template
      sync to both Claude and Cursor skill layouts.

## 2. Users & Motivation

**Primary user: the operator planning the next change.** "I want to drop
notes about a feature and have you help me plan… back and forth until I am
happy with the plan." The skills make that conversation produce the
strategy assessment, the design artifact, and machine-ready criteria as
byproducts — so lighting the campfire needs no re-translation.

**Secondary user: the verification gate and reviewers.** UX criteria in the
tiered format give the verifier agent concrete states to check in the
sandbox, closing the gap where cross-surface UX regressions were caught
only by the human smoke.

## 3. Approach

Three skill templates plus two template hooks, all in team-setup (upstream,
generic), configured per workspace:

- **Grounding config** — a single workspace file declaring where judgment
  inputs live: objective docs, design method, tokens, glossary. Skills read
  the config, then the docs; a missing declaration is a reported gap.
- **`/strategize`** — structured assessment: fit against declared
  objectives, related ideas, risks, user value, bug-retro hooks (for bug
  intake: what the report teaches the process), recommendation with shape.
  Output is a markdown assessment block suitable for pasting into the
  intake issue or PRD §1/§2.
- **`/design-ux`** — options → recommendation → artifact: proposes 2–3
  expressions of the value, renders the chosen one as an artifact file in
  the docs repo (fidelity proportional to risk), and emits UX acceptance
  criteria in tiered format for direct inclusion in the spec.
- **Template hooks** — spec template gains a UX criteria subsection;
  PRD template §3 gains an optional design-artifact link line;
  `/write-prd` gains the right-sizing checkpoint text.
- **First consumer configuration** — a consumer-side config change (consumer
  workspace repo) pointing at its workbook, the Rosetta foundations, and
  website-design-method docs; no consumer content upstream.

## 4. Data Contracts

```ts
// Workspace grounding config (consumer-owned, e.g. .rosetta/grounding.json)
interface GroundingConfig {
  objectives: string[]; // business objective / north-star docs
  designMethod?: string[]; // method + token docs
  glossary?: string[];
  foundations?: string[]; // philosophy/constitution docs
}

// Strategy assessment (skill output shape, markdown-rendered)
interface StrategyAssessment {
  intakeRef: string; // issue / note / queue item
  fit: string; // against cited objectives (with doc refs)
  recommendation: "proceed" | "improve" | "park" | "decline";
  shape?: string; // when proceeding: suggested scope
  risks: string[];
  missing: string[]; // unanswered questions, undeclared objectives
  bugRetro?: string; // for bug intake: process lesson
}

// UX criteria block (flows into spec unchanged)
interface UxCriteria {
  artifactRef: string; // path to sketch/wireframe/comp
  criteria: Array<{
    tier: "test" | "agent" | "manual";
    text: string; // states, empty/error states, responsive behavior
  }>;
}
```

## 5. Constraints & Dependencies

- Platform boundary: skill templates and template hooks are upstream
  team-setup artifacts; grounding configs and all judgment content are
  consumer-owned. The consumer config lands as a separate consumer change.
- Human-in-the-loop invariant: outputs are advisory artifacts for the
  planning conversation; nothing here bypasses or automates the spec +
  envelope Approve.
- UX criteria must render in the existing tiered criteria format so
  `sdlc-workflow` verification and `prd-lint`/spec validation need no new
  parser (any validation change is a small engine dependency, coordinated
  with the spec-format lint work in the envelope-integrity spec).
- Distribution through the existing team-setup sync (both `.claude/` and
  `.cursor/` layouts), same as current skills.
- Skills never pull application data; grounding is docs-only.

## 6. Risks & Open Questions

- Skill outputs are only as good as the declared grounding docs — a consumer's
  workbook has known `OPEN:` gaps; the flag-the-gap behavior is the
  mitigation, and the first strategist runs will mostly surface
  documentation debt (that is the point).
- Right-sizing guidance risks being ignored under deadline pressure; it is
  advisory by design, with decompose's spec review gate as the backstop.
- Artifact conventions for UX (where sketches live, naming) need one
  decision per consumer docs repo; the config carries it.

## 7. Rollout & Phases

1. **Phase 1 — Grounding config + `/strategize`:** config schema and
   loader convention, strategist skill template, distribution through
   team-setup sync; first consumer config as initial grounding.
2. **Phase 2 — `/design-ux` + template hooks:** UX skill template, spec
   template UX-criteria section (validation coordinated), PRD template
   artifact link, artifact placement convention.
3. **Phase 3 — PM right-sizing checkpoint:** `/write-prd` and
   `/write-bug-spec` gain the scope sanity check; decompose prompt
   references the same guidance so both sides of the gate agree.

## 8. Future Considerations

- Strategy assessments recorded as Chronicle artifacts so recommendations
  build a track record like gate verdicts.
- Intake integration: the consumer ticket-intake pipeline (PRD-0007) invoking
  `/strategize` output as the normalized assessment attached to the issue.
- Design-tool integrations (Figma export, Storybook stubs) once artifact
  conventions stabilize.
