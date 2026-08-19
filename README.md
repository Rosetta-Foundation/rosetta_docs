# rosetta_docs

The durable, versioned home for **cross-cutting Rosetta artifacts** — the "what and
why" of the platform that doesn't belong to any single code repo.

> Chronicle is the memory. Wayfinder is the guide. This repo is the record of intent.

## Contents

| Folder                          | Holds                                                                                                    |
| ------------------------------- | -------------------------------------------------------------------------------------------------------- |
| [`foundations/`](foundations)   | The constitution — founding context, manifesto, principles, glossary, settled decisions. **Read first.** |
| [`product/`](product)           | Product Requirements Documents (PRDs) — capabilities framed **before** they are built.                   |
| [`architecture/`](architecture) | Architecture Decision Records (ADRs) — decisions already made.                                           |
| [`process/`](process)           | Claim hygiene — research tiers, epistemic status, adversarial review, historical parallel (Proposed).    |
| [`docs/`](docs)                 | Cross-cutting product & workspace docs (vision, origin story, [website branding options](docs/WEBSITE-BRANDING.md), assets). |
| [`story/`](story)               | Long-form essay drafts developing the civilizational philosophy behind Rosetta.                          |
| [`guides/`](guides)             | Communication sketches at story and field-guide altitude (one-page explanation, question-driven guide).  |
| [`shared/`](shared)             | Shared assets referenced across Rosetta repos.                                                           |

## Why a dedicated repo

These artifacts are first-class organizational knowledge: they need history, review,
and a single source of truth. Previously they were scaffolded as frozen copies into
each engineer's workspace root by `team-setup`, which meant no shared history and
silent divergence. `team-setup` now **clones** this repo at the workspace root
instead — one versioned home, PR-reviewable, decoupled from any component.

## Conventions

- **PRDs** — see [`product/README.md`](product/README.md). Filename `PRD-NNNN-short-kebab-title.md`; register each in the product Records table.
- **ADRs** — see [`architecture/README.md`](architecture/README.md).
- **Load-bearing claims** in prose — see [`process/README.md`](process/README.md) (research
  grounding, epistemic status, adversarial review). SDLC gates cover ships; process covers writing.
- Every artifact is authored to be legible to **both humans and machines**: structured
  frontmatter, list-based content, checkbox acceptance criteria, explicit links.
- **Git:** Conventional Commits via husky. Default is `f/` / `b/` topic branches + PR — do not
  commit on `main` unless a human authorizes a documented exception (foundation bootstrap or
  emergency hotfix; see the workspace root `CLAUDE.md`).

## Workspace placement

Cloned at the Rosetta workspace root beside the code repos:

```
rosetta/
├── rosetta_docs/         ← this repo (product, architecture, docs, shared)
├── rosetta_chronicle/
├── rosetta_wayfinder/
└── rosetta_dev-scripts/
```

## License

All content in this repo is licensed [CC BY 4.0](LICENSE) — share and adapt with attribution.
Copyright 2026 Rosetta Foundation.
