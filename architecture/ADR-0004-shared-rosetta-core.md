# ADR-0004: Shared Rosetta Core — What's Shared vs. Per-App

**Status:** Accepted

**Date:** 2026-07-24

---

> Contracts and pure logic are shared. Repository implementations are per-app.
> Extract the shared core deliberately — after the demo, or at the first
> divergence bug — never speculatively mid-sprint.

---

# Background

Rosetta now has more than one application consuming the same knowledge domain:

- **Chronicle** — the Node CLI engine (`rosetta_chronicle`) that captures and
  structures engineering activity.
- **Wayfinder** — the Tauri desktop app (`rosetta_wayfinder`, see
  [PRD-0010](../product/PRD-0010-wayfinder-local-app.md)) that gives non-engineers
  the same capture/query capabilities.

Both apps operate on the **same on-disk formats**: the chronicle directory layout,
the structured sidecar, and — imminently, once Wayfinder issue #5 lands — the
`queue.md` work-queue format defined by [PRD-0007](../product/PRD-0007-personal-work-queue.md).

This raised the question during review of Wayfinder's `workspace.service.ts`: **what
should be shared between the apps, and what should stay separate?**

[ADR-0002](ADR-0002-personal-vs-organizational-chronicle.md) already anticipated the
destination — "`rosetta_chronicle` is the **engine** — machinery only, published as a
package and consumed everywhere." This ADR defines the concrete boundary and the
timing.

# Decision

## The sharing line: contracts and pure logic are shared; implementations are not

The split falls cleanly along the Handler / Service / Repository layers:

| Layer | Shareable? | Rationale |
|-------|-----------|-----------|
| **Types / contracts** (`QueueItem`, `QueueRef`, `Activity`, `Evidence`, chronicle-format types) | ✅ **Shared** | Pure data. No platform dependency. Divergence here is a silent interop bug. |
| **Pure logic** (`queue.utils.ts` parse/serialize/prioritize, `tags.utils.ts`) | ✅ **Shared** | Deterministic, dependency-free functions. Both apps must produce byte-identical output on the same files. |
| **Repository interfaces** (`IGitRepository`, `IQueueStore`) | ✅ **Shared** | The contract is common even when the implementation differs. |
| **Repository implementations** | ❌ **Per-app** | Chronicle shells out to the `git` binary via Node `child_process` + Node `fs`. Wayfinder calls libgit2 across the Tauri IPC boundary + Rust fs. Same interface, fundamentally different transport (see [ADR-0003](ADR-0003-tauri-thin-rust-core.md)). |
| **Services** | ➗ **Case-by-case** | Domain logic that composes only shared repositories *can* be shared; app-specific orchestration (e.g. Wayfinder's `WorkspaceService` first-run provisioning) stays per-app. |
| **Handlers** | ❌ **Per-app** | Entry points are inherently platform-specific (CLI args vs. UI events vs. Tauri commands). |

**The load-bearing case is the format parsers.** Both apps read and write `queue.md`
and the chronicle sidecar. If Wayfinder's parser and Chronicle's `queue.utils.ts`
drift, interop breaks silently: an engineer edits `queue.md` via the CLI, a PM opens
it in Wayfinder, and the two disagree about its contents. Keeping the parser + its
types single-sourced is a **correctness** requirement, not a DRY preference.

## The timing: extract deliberately, not speculatively

**Do NOT extract a shared package during the Wayfinder demo sprint.** Reasons:

- The *present* overlap is small — as of this ADR, Wayfinder does not yet have
  `QueueItem` or the queue parser (they arrive with issue #5). Extracting a package
  now is monorepo/versioning plumbing for near-zero current benefit.
- Premature extraction couples two fast-moving repos and adds release friction
  mid-sprint.

**Instead, during the sprint:**

1. When Wayfinder issue #5 needs the queue parser, **copy `queue.utils.ts` + the
   queue types verbatim** from Chronicle — do not rewrite them. Identical source now
   makes mechanical extraction trivial later.
2. **Quarantine shared-destined code** in a dedicated folder in each app
   (`src/shared/` or `src/core/`) so future extraction is lift-and-shift, not
   archaeology.
3. Add a comment at the top of any copied file noting its origin and that it is
   slated for `@rosetta/core`.

**Extract `@rosetta/core` as a deliberate step when EITHER trigger fires:**

- The Wayfinder demo has shipped and the extraction can be done without sprint
  pressure, **or**
- The duplicated parser causes its first divergence bug (the "real pain" trigger).

Whichever comes first. Extraction is never justified by speculation alone.

## The eventual shape

The destination (already committed to by ADR-0002) is a neutral core package:

```
@rosetta/core                     (published package — the "engine")
  types/          QueueItem, Activity, Evidence, chronicle-format contracts
  utils/          queue parse/serialize/prioritize, tag inference (pure)
  contracts/      IGitRepository, IQueueStore, … (interfaces only)

rosetta_chronicle   (consumer)    Node repository impls + CLI handlers + services
rosetta_wayfinder   (consumer)    Tauri repository impls + UI handlers + services
```

Each app supplies its own repository *implementations* behind the shared interfaces.
The engine holds no platform code and no I/O — it is machinery only, exactly as
ADR-0002 framed it.

# Consequences

**Positive:**

- One source of truth for the on-disk formats → no silent interop drift between CLI
  and app.
- The shared/per-app boundary is explicit, so contributors and agents know where new
  code belongs.
- Deferring extraction keeps the demo sprint fast; quarantining now keeps the later
  extraction cheap.
- Aligns with and makes concrete the "engine consumed everywhere" decision of
  ADR-0002.

**Negative / costs:**

- A window of **deliberate duplication** exists between issue #5 and the eventual
  extraction. Mitigated by copying verbatim and quarantining — the duplication is
  known, bounded, and scheduled, not accidental.
- Requires discipline: the copied parser must be updated in lockstep if it changes
  before extraction. A divergence is the signal to extract, not to patch twice.

**Rule of thumb:**

> If it is data or a pure function on that data → it belongs in the shared core
> (copied verbatim until extracted). If it touches a platform (fs, git binary, IPC,
> UI) → it is a per-app implementation behind a shared interface.

# Scope

Governs all Rosetta applications that consume the chronicle/queue domain. Extends
ADR-0002 (engine vs. repository) and composes with ADR-0003 (thin Rust core — the
reason Wayfinder's repository implementations must differ from Chronicle's).
