# ADR-0006: TypeScript 7 + Bun Toolchain

**Status:** Accepted

**Date:** 2026-07-31

---

> TypeScript 7 (native compiler) type-checks and builds. Bun installs. tsx runs
> dev entrypoints. @swc/jest transpiles tests, with type-checking owned by the
> build. Yarn 1, ts-node, and ts-jest are retired across all Rosetta repos.

---

# Background

Rosetta standardized early on Yarn 1 + tsc (JS) + ts-node + ts-jest. By
mid-2026 every element of that stack had a categorically faster successor:

- **TypeScript 7.0** (GA July 8, 2026) rewrote the compiler in Go: full builds
  run 8–12x faster with parallel type-checking, under the same `tsc` entry
  point and `typescript` npm package.
- **Bun 1.3** installs cold ~5x faster than pnpm and ~8x faster than Yarn 1;
  warm installs are near-instant. Yarn 1 is in maintenance and the slowest
  mainstream option.
- **ts-jest** type-checks every file on every test run — usually the largest
  recurring cost in a TS workflow. **@swc/jest** transpiles only (Rust),
  leaving type-checking to the now-fast native `tsc`.
- **ts-node** depends on the TypeScript programmatic compiler API, which TS7
  does not ship until 7.1 (~October 2026). **tsx** (esbuild-based) has no such
  dependency and is faster.

Build speed is a developer-loop concern ("lightning fast" is an explicit
workspace priority), and the migration was measured before adoption.

# Decision

All Rosetta repos use, and `team-setup` scaffolds:

| Concern               | Tool                                    | Replaces                |
| --------------------- | --------------------------------------- | ----------------------- |
| Package manager       | **Bun 1.3+** (`bun install`, `bun run`) | Yarn 1                  |
| Type-check + emit     | **TypeScript 7** (`tsc`, native)        | TypeScript 5 (JS `tsc`) |
| Dev-mode TS execution | **tsx**                                 | ts-node                 |
| Jest transform        | **@swc/jest** (transpile-only)          | ts-jest                 |
| Runtime               | **Node 20+** (unchanged)                | —                       |

Division of labor: **type-checking happens exactly once, in the build**
(`tsc`); tests and dev execution transpile without checking. Node remains the
runtime for CLIs, jest, and Vite — Bun is adopted as package manager only.

## Required config migrations (TS7)

- `moduleResolution: "node"` (node10) is removed → `"nodenext"` for Node
  CLIs, `"bundler"` for Vite apps.
- `types` now defaults to `[]` → list globals explicitly
  (`"types": ["node", "jest"]`).
- `noUncheckedSideEffectImports` is default-on → side-effect imports (e.g.
  CSS in Vite apps) need ambient module declarations.
- swc transforms mirror the decorator flags: `legacyDecorator` +
  `decoratorMetadata` — required by the InversifyJS HSR pattern
  (`experimentalDecorators`/`emitDecoratorMetadata` are ported in TS7's
  native emit and verified by each repo's test suite).

## Measured results (this workspace, 2026-07-31)

| Repo                | Build (before → after)                    | Tests                                                                                |
| ------------------- | ----------------------------------------- | ------------------------------------------------------------------------------------ |
| rosetta_chronicle   | ~1.3s → ~0.15s                            | 205/205 pass; transform CPU −40%; wall unchanged (dominated by e2e subprocess suite) |
| rosetta_wayfinder   | 2.45s → 1.32s (tsc portion ~1.9s → ~0.8s) | 78/78 pass; 0.81s → 0.49s                                                            |
| rosetta_dev-scripts | sub-second                                | 105/105 pass                                                                         |

Clean `bun install` per repo: ~1.3–1.7s (was tens of seconds under Yarn 1).

# Consequences

**Positive:**

- Type-check feedback is effectively instant on repos of this size; installs
  are no longer a noticeable step.
- One type-check per pipeline instead of re-checking in every test run.
- CI workflows simplify to `setup-node` + `setup-bun` + `bun install --frozen-lockfile`.

**Negative / costs:**

- **TS7 has no programmatic compiler API until 7.1.** Tools that consume the
  compiler as a library (typescript-eslint, ts-morph, template checkers)
  need the `@typescript/typescript6` side-by-side package until then. No
  Rosetta repo currently depends on such tooling.
- **`emitDecoratorMetadata` on the native compiler is a verify-don't-assume
  path** ecosystem-wide. Exposure is low (HSR injects by explicit
  `@inject(TOKEN)` symbols, not inferred types) and every repo's DI test
  suite passes; any future decorator-heavy dependency must be re-verified.
- Bun postinstall scripts are blocked by default; native-binding packages
  (`@swc/core`, `unrs-resolver`) are allowed via `trustedDependencies`.
- Wearing two runtimes (Bun for installs, Node for execution) is a mild
  cognitive tax; scripts stay `bun run <script>` for uniformity.

**Out of scope / unchanged:** Wayfinder's Tauri release build remains
cargo/Rust-bound — no JS toolchain affects it. Mitigations there are
ADR-0003's thin-Rust-core stance plus incremental builds/sccache.

# Scope

Applies to every Rosetta repo with a `package.json`. The scaffolding
(`rosetta_dev-scripts` team-setup: prerequisites, installs, bootstrap,
root templates, agent permission allowlists) enforces and propagates it.
Supersedes the "always use yarn" workspace policy previously recorded in the
root `CLAUDE.md` template.
