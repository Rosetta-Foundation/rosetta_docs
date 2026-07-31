---
id: PRD-0013
title: Markdown Rendering in Ask Wayfinder Responses
status: Proposed
date: 2026-07-27
owner: Russ Watson
related_adrs: [ADR-0003]
related_specs: []
supersedes: null
---

# PRD-0013: Markdown Rendering in Ask Wayfinder Responses

> Wayfinder's chat responses render as formatted HTML instead of raw markdown text.

## 1. Overview & Goals

### 1.1 Purpose

Claude returns markdown-formatted text (headings, bold, lists, code spans, links) in
`KnowledgeAnswer.answer` and `ChatTurnResult.reply`. Today the UI renders that text verbatim inside
a `white-space: pre-wrap` `<p>` — `##`, `**`, and `` ` `` characters show up literally instead of
rendering as structure. This is most visible in the "Ask Wayfinder" org-knowledge mode, where
answers routinely include headings and bullet lists (see the AI Ops IPM meeting example — a
`## AI Ops IPM Meeting` heading rendered as literal `##` text).

### 1.2 Goals

- Chat responses (`Chat.tsx`) render markdown as HTML: headings, bold/italic, lists, code
  spans/blocks, links, and blockquotes.
- Rendering happens client-side, at display time — the stored `answer`/`reply` strings remain raw
  markdown (unchanged data contract).
- Visual result matches Wayfinder's existing dark theme (and the light theme introduced by
  PRD-0014) — not a generic unstyled markdown block.

### 1.3 Non-Goals

- No markdown _authoring_ — the user's own input (the textarea) stays plain text. Only assistant
  responses render as markdown.
- No support for embedded images, tables, or footnotes in v1 — Claude responses observed so far
  don't use them; add if they show up.
- Not changing citations rendering (`chat__citations` list) — that stays as-is.

### 1.4 Acceptance Criteria

- [ ] A response containing `## Heading`, `**bold**`, `` `code` ``, and a `-` list renders as an
      `<h2>`, `<strong>`, `<code>`, and `<ul><li>` respectively — not literal characters.
- [ ] Markdown rendering is sanitized — no raw HTML from the model can execute script or inject
      arbitrary DOM (the model output is untrusted content, per the retrieval grounding in
      PRD-0010 §3.3/Phase 2).
- [ ] Rendered output is visually consistent with the panel's existing typography (font, color
      variables `--text`/`--muted`/`--accent`, spacing) in both dark and light themes.
- [ ] `yarn lint` (tsc --noEmit) and `yarn test` pass with no regressions.

## 2. Users & Motivation

Anyone using "Ask Wayfinder" — PMs, executives, engineers — reads assistant responses as their
primary output. Structured answers (meeting summaries, action-item lists, PRD status) are far more
scannable as real headings and lists than as a wall of text with stray `#` and `*` characters. This
directly affects demo credibility: a raw `##` in front of stakeholders reads as broken, not as a
formatting quirk.

## 3. Approach

Use the standard `unified`/`remark`/`rehype` pipeline (`remark-parse` → `remark-gfm` →
`remark-rehype` → `rehype-sanitize` → `rehype-stringify`) to convert markdown to a sanitized HTML
string, then render it via `dangerouslySetInnerHTML`.

Wayfinder doesn't use Tailwind, so styling is hand-rolled CSS instead of `prose`.

- **Library choice:** `unified` + `remark-parse` + `remark-gfm` + `remark-rehype` +
  `rehype-sanitize` + `rehype-stringify`. This battle-tested pipeline runs entirely client-side
  (no build-time step needed), and `rehype-sanitize` is non-negotiable — Claude's raw text is
  untrusted model output and must never be interpreted as live HTML.
- **Where conversion happens:** a pure utility `src/utils/markdown.ts` exporting
  `renderMarkdown(text: string): string` (sync, returns sanitized HTML string). Pure function per
  the workspace `src/utils/` convention — no DI needed, it wraps a stateless library call.
- **Where it's used:** `Chat.tsx`'s message rendering replaces
  `<p className="chat__content">{m.content}</p>` with
  `<div className="chat__content" dangerouslySetInnerHTML={{ __html: renderMarkdown(m.content) }} />`
  for `role === 'assistant'` messages only. User messages (`role === 'user'`) keep the current
  plain-text rendering — nothing to convert, and it avoids ever rendering user-typed content as HTML.
- **Styling:** new CSS rules scoped under `.chat__content` for `h1`–`h4`, `p`, `ul`/`ol`, `li`,
  `code`, `pre`, `blockquote`, `a` — using the existing CSS variables (`--text`, `--muted`,
  `--accent`, `--border`) so it inherits whichever theme (dark/light, PRD-0014) is active.

## 4. Data Contracts

No changes to `KnowledgeAnswer`, `ChatTurnResult`, or any IPC/service boundary type. This is
presentation-only — `answer`/`reply` remain raw markdown strings at every layer.

```ts
// src/utils/markdown.ts
export const renderMarkdown = (markdown: string): string; // returns sanitized HTML
```

## 5. Constraints & Dependencies

- Pure function lives in `src/utils/`, per the workspace HSR convention — this is presentation
  logic, not a Service; it does not touch the DI container.
- New runtime dependencies: `unified`, `remark-parse`, `remark-gfm`, `remark-rehype`,
  `rehype-sanitize`, `rehype-stringify`. Added via `yarn add` (Yarn per workspace convention).
- `rehype-sanitize`'s default schema is the security boundary for untrusted model output — do not
  loosen it to allow raw `<script>`/`<style>`/event-handler attributes.
- No change to ADR-0003 (Rust stays a logic-free transport shim) — this is pure frontend TypeScript.

## 6. Risks & Open Questions

- **Bundle size:** `unified` + plugins add real weight to the frontend bundle. Acceptable for a
  local-first Tauri app (no network-transfer cost), but worth noting if Wayfinder ever ships a
  web build.
- **Sanitization gaps:** `rehype-sanitize`'s default schema covers standard elements; verify it
  rejects `javascript:` URLs in `a href` and any inline event handlers before shipping.
- **Streaming responses:** if streaming replies are added later (not in scope today), markdown
  parsing mid-stream produces flickering/invalid partial HTML — that's a future-phase problem, not
  this PRD's.

## 7. Rollout & Phases

1. **Phase 1** — Add the remark/rehype pipeline, `renderMarkdown` utility, wire into `Chat.tsx` for
   assistant messages only, add scoped CSS for rendered markdown elements in both themes.

## 8. Future Considerations

- Extend markdown rendering to the Standup panel (`Standup.tsx`) and today's-note preview, which
  render similarly-shaped Claude output as plain text today.
- Tables and footnote support if Claude responses start using them (add `remark-gfm` already
  covers tables — verify once in place).
