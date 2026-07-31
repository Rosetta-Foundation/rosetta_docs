---
id: PRD-0014
title: Light & Dark Theme
status: Proposed
date: 2026-07-27
owner: Russ Watson
related_adrs: [ADR-0003]
related_specs: []
supersedes: null
---

# PRD-0014: Light & Dark Theme

> Wayfinder supports both a light and a dark theme, following the OS preference by default.

## 1. Overview & Goals

### 1.1 Purpose

Wayfinder currently ships one hardcoded dark theme (`src/ui/styles.css` `:root` block: `--bg:
#0f1115`, `--text: #e6e8ec`, etc.). Some users — including executive/PM demo audiences — prefer or
expect a light theme, and OS-level dark/light preference should be respected rather than ignored.

### 1.2 Goals

- A light theme exists alongside the current dark theme, using the same CSS variable set already
  defined at `:root` in `styles.css` — no component restyling needed beyond variable values.
- Default theme follows the OS-level preference (`prefers-color-scheme`).
- User can manually override the OS default and the choice persists across app restarts.
- Every existing panel (Queue, Standup, Capture, Chat/Ask Wayfinder, Chronicle ledger, Onboarding)
  reads correctly in both themes — no hardcoded colors bypass the variable system.

### 1.3 Non-Goals

- No per-panel theming — one theme applies app-wide.
- No custom/user-defined color themes — light and dark only, in v1.
- No redesign of layout, spacing, or component structure — this is a color-token swap, not a visual
  redesign.

### 1.4 Acceptance Criteria

- [ ] Launching Wayfinder on a system set to light mode shows the light theme by default; dark
      system preference shows dark by default.
- [ ] A visible toggle lets the user switch theme regardless of OS preference.
- [ ] The chosen theme persists in `~/.wayfinder/config.json` (the same file that already stores
      `chroniclePath`) and is restored on next launch.
- [ ] Every current screen — Onboarding, boot/loading, error state, and the five main panels —
      renders with sufficient contrast in both themes (no light-text-on-light-bg or
      dark-text-on-dark-bg regions).
- [ ] `yarn lint` passes; no new hardcoded hex colors introduced in component code (colors go
      through CSS variables only).

## 2. Users & Motivation

Executives and PMs reviewing Wayfinder in a demo or day-to-day may simply prefer light mode, or may
be presenting on a shared/projector setup where a dark UI reads poorly. Respecting OS preference is
also a baseline expectation for any modern desktop app — its absence today reads as unfinished.

## 3. Approach

The existing `styles.css` already centralizes every color as a `:root` custom property
(`--bg`, `--panel`, `--text`, `--muted`, `--accent`, `--error`, `--border`). This is the right
foundation — theming becomes swapping the values of that one variable block based on a `data-theme`
attribute, not rewriting component styles.

- **Theme values:** define two variable sets — the current dark palette stays as `:root` /
  `[data-theme="dark"]`, and a new `[data-theme="light"]` block with light-appropriate values
  (light `--bg`/`--panel`, dark `--text`, adjusted `--accent` contrast, lighter `--border`).
- **Detection:** on boot, read `window.matchMedia('(prefers-color-scheme: dark)')` for the OS
  default when no persisted preference exists.
- **Persistence:** extend `WayfinderConfig` (already in `types.ts`, already read/written by
  `ConfigRepository` per PRD-0010) with an optional `theme?: 'light' | 'dark' | 'system'` field.
  `'system'` (the default) means "follow OS preference"; `'light'`/`'dark'` is an explicit
  override.
- **Application:** a small `useTheme()` hook (or equivalent logic in `App.tsx`) sets
  `document.documentElement.dataset.theme` on boot and whenever the user toggles it; persists via
  the existing config read/write path (`ConfigRepository` → `changeChronicle`-style round trip, or
  a new lightweight `saveThemePreference` on the same repository).
- **Toggle UI:** a small icon button in the `.brand` header (sun/moon) that cycles
  system → light → dark, consistent with the existing header layout (next to the logo/tagline).

## 4. Data Contracts

```ts
// src/types.ts — extend existing WayfinderConfig
export interface WayfinderConfig {
  chroniclePath?: string;
  /** 'system' follows OS preference; explicit values override it. Defaults to 'system'. */
  theme?: 'system' | 'light' | 'dark';
}
```

No changes to any IPC command signature — `theme` is just another field in the existing
`config.json` read/write via `ConfigRepository` (`IPC.configPath`, `readFile`/`writeFile`).

## 5. Constraints & Dependencies

- Must build on the existing CSS-variable architecture in `styles.css` — no per-component color
  literals should be introduced; anything not already using a `var(--...)` should be migrated as
  part of this work.
- Persistence reuses the existing `~/.wayfinder/config.json` mechanism (`ConfigRepository`,
  established in PRD-0010) rather than introducing a new storage location.
- Follows the HSR pattern: any new persistence logic belongs in `ConfigRepository`
  (`IConfigRepository`), consumed by `WorkspaceService` or a small new `ThemeService` if the logic
  grows beyond a trivial read/write — not inlined in `App.tsx`.
- No new native/Rust surface — reading `prefers-color-scheme` and setting a `data-theme` attribute
  are both pure web APIs; ADR-0003's Rust-is-transport-only boundary is unaffected.

## 6. Risks & Open Questions

- **Contrast audit:** every existing hardcoded assumption (e.g. `.btn` text color `#1a1205` against
  `--accent`) needs a check against the light palette's accent color — open question on final light
  palette values, to be resolved during implementation with a manual pass over every panel.
- **Markdown rendering (PRD-0013) interaction:** rendered markdown in chat responses must also use
  CSS variables, not fixed colors, so it themes correctly — sequence PRD-0013 styling to depend on
  this PRD's variable set (or land them together).
- **System-preference change while running:** decide whether Wayfinder should live-update when the
  OS theme changes mid-session (via a `matchMedia` change listener) or only re-check on next
  launch. Recommend live-update for correctness, since it's a small addition.

## 7. Rollout & Phases

1. **Phase 1** — Add the light `[data-theme="light"]` variable block, OS-preference detection, and
   a manual toggle in the header; audit and fix any non-variable color usage across all panels.
2. **Phase 2** — Persist the user's explicit choice to `config.json` via `ConfigRepository`; restore
   on boot before first paint (avoid a flash of the wrong theme).

## 8. Future Considerations

- Additional theme variants beyond light/dark (e.g. a high-contrast mode) if accessibility needs
  surface later.
- Per-OS accent-color integration (e.g. matching macOS system accent color) — deferred, not
  requested.
