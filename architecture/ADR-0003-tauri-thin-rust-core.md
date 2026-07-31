# ADR-0003: Tauri Apps — Thin Rust Core, Business Logic in TypeScript

**Status:** Accepted

**Date:** 2026-07-24

---

> All logic that thinks lives in TypeScript under Handler / Service / Repository.
> The Rust core is a thin, logic-free transport shim.

---

# Background

Wayfinder (see [PRD-0010](../product/PRD-0010-wayfinder-local-app.md)) is the first
Rosetta application built as a **Tauri** desktop app rather than a Node CLI or
service. Tauri applications are structurally split into two processes:

- A **web frontend** (React + TypeScript) rendered in a system webview.
- A **native core** that must be written in **Rust** — this is the framework's
  runtime. It is the only layer with direct access to the filesystem, native
  libraries (libgit2), OS APIs, and outbound network/SDK calls.

The two communicate across Tauri's IPC boundary: the frontend calls Rust functions
registered as "commands" via `invoke('command_name', args)`.

This raises a tension with the mandatory Rosetta architecture standard, which
requires **all TypeScript** to follow Handler / Service / Repository + InversifyJS
(see `../.claude/rules/architecture-hsr.md`). Tauri makes "all TypeScript" literally
impossible — some code _must_ be Rust. The question is: **where do we draw the line,
and how do we keep HSR intact?**

---

# Decision

**The Rust core is kept as thin as physically possible — a logic-free transport
shim. All business logic lives in TypeScript, following the standard Rosetta HSR +
InversifyJS pattern.**

### The Rust core contains ONLY:

- Tauri command registrations that expose **primitive, mechanical operations** to
  the frontend: `git_commit`, `git_log`, `read_file`, `write_file`,
  `claude_invoke`, and the like.
- No business logic, no orchestration, no validation beyond what the native call
  itself requires. Think **driver**, not **service**.
- Each command is a near-1:1 wrapper over a native capability (a libgit2 call, a
  filesystem call, an HTTPS call to the Claude API).

### The TypeScript frontend contains ALL logic, in full HSR:

```
Frontend (TypeScript — HSR + InversifyJS)
  Handler     → UI event handlers / Tauri command dispatch. No business logic.
  Service     → business & orchestration logic (chronicle synthesis, queue
                operations, RAG orchestration, promotion flow). Composes repositories.
  Repository  → resource access. Wraps the thin Rust commands via Tauri `invoke()`.
                A GitRepository calls invoke('git_commit', …) instead of shelling out.
       │
       ▼  (Tauri IPC boundary)
Rust core (thin transport — no logic)
  git_commit · git_log · read_file · write_file · claude_invoke · …
```

A Repository in TypeScript treats `invoke('git_commit', …)` exactly as any other
repository treats an SDK or HTTP call: it is the resource-access seam. The rest of
the pattern is unchanged from Chronicle — `@injectable()` classes, `@inject(TOKEN)`
constructor injection, `Symbol.for` tokens, interface + implementation co-located,
composition root wiring the container.

### The one deliberate exception: credential-bearing calls stay in Rust

Calls that carry secrets — specifically the **Claude API HTTP call** — are
implemented in the Rust core, not the frontend. `ANTHROPIC_API_KEY` lives in the
native process and is **never exposed to the webview**. The frontend reaches
Claude through a thin TypeScript `ClaudeRepository` that wraps
`invoke('claude_invoke', …)`.

This is a security-posture decision, not a logic decision: the _what to ask and how
to use the answer_ (business logic) stays in the TS Service; only the _credentialed
transport_ lives in Rust.

---

# Consequences

**Positive:**

- The mandatory HSR + InversifyJS standard holds everywhere it can — every piece of
  code that reasons, orchestrates, or validates is TypeScript in the familiar
  pattern. Reviewers and future agents apply the same mental model as in Chronicle.
- The Rust surface is minimal and mechanical, so it needs little Rust expertise to
  maintain and is low-risk to review.
- Secrets (API keys) never cross into the webview — a stronger security
  posture than routing Claude API calls from the frontend.
- Business logic is testable with the existing Jest + container-per-test approach;
  the Rust shim is thin enough to test with a handful of Rust unit tests.

**Negative / costs:**

- There is a hard architectural seam (the IPC boundary) that TypeScript HSR cannot
  cross. Repositories that wrap `invoke` are the only place this leaks, and that is
  by design — it is the resource-access layer, which is exactly where a boundary
  belongs.
- Two languages in one app: contributors need enough Rust to add a new primitive
  command when a genuinely new native capability is required. This is bounded — new
  commands are rare and formulaic.
- Discipline required: the temptation to "just add a little logic in Rust because
  it's closer to the data" must be resisted. Logic in Rust is a violation of this
  ADR, treated the same as an HSR violation.

**Rule of thumb for where code goes:**

> If it makes a decision, transforms data, or orchestrates — it is TypeScript in a
> Service. If it is a single mechanical native operation or carries a credential —
> it is a thin Rust command. When in doubt, it goes in TypeScript.

---

# Scope

This ADR governs **all Tauri applications in Rosetta**, current and future. Wayfinder
is the first; the pattern is the standard for any subsequent desktop app.

It **extends** — does not replace — the HSR + InversifyJS rule in
`../.claude/rules/architecture-hsr.md`. That rule remains in force for all
TypeScript. This ADR only defines how that rule coexists with Tauri's mandatory Rust
layer.

The operational conventions for building against this pattern are captured as a
skill (`tauri-hsr`) so all agents apply it consistently.
