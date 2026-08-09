---
id: PRD-0018
title: Siri → Chronicle Voice Log Capture
status: Draft
date: 2026-08-02
owner: Russ Watson
related_adrs: [ADR-0002, ADR-0005]
related_specs: []
supersedes: null
---

# PRD-0018: Siri → Chronicle Voice Log Capture

> Speak “New Chronicle log entry,” pause, dictate a personal journal note — and
> have it land in the personal Chronicle without opening an editor.

## 1. Overview & Goals

### 1.1 Purpose

Wayfinder voice ([PRD-0017](PRD-0017-wayfinder-voice-audio-surface.md)) covers
Ask and dictation **inside** the app. The everyday capture moment is different:
phone in pocket, hands busy, no app open — “Siri. New Chronicle log entry.”
then a brief pause and spoken journal content. That OS wake path does not exist
today; notes and activity stay keyboard- or in-app-bound.

This PRD owns the **Apple capture surface** (Shortcuts / later App Intents) and
a **personal capture bridge** that posts into existing Chronicle contracts
([PRD-0003](PRD-0003-notes-as-authoritative-input.md) notes append;
[PRD-0015](PRD-0015-chronicle-activity-schema-and-open-ingestion.md) inbox). It
does not invent a new ledger format.

### 1.2 Goals

- Let a human invoke a fixed Siri/Shortcuts phrase, pause, dictate a personal
  log, and have the transcript land in their **personal** Chronicle without
  opening an editor or copying text.
- Stay local-first: transcript stays on-device until the user’s own capture
  bridge; no third-party journal SaaS; no multi-tenant hosted ingest required
  ([ADR-0005](../architecture/ADR-0005-decentralized-by-construction.md)).
- Reuse Chronicle storage contracts — notes append first; activity envelopes
  when open ingestion ships — with clear provenance (`[Siri]` / `ext:rosetta/siri-log`).

### 1.3 Non-Goals

- Not a general voice assistant or open-web Siri replacement (same line as
  PRD-0017).
- Not Wayfinder Ask-by-voice or in-app note dictation (PRD-0017).
- Not organizational promotion of voice logs — knowledge stays personal until
  the promotion gate ([ADR-0002](../architecture/ADR-0002-personal-vs-organizational-chronicle.md),
  [PRD-0006](PRD-0006-artifact-capture-and-promotion.md)).
- Not a clinical / PHI documentation channel (Comita healthcare guardrails).
- Not Apple Watch or CarPlay as Phase 1 deliverables (see Future).

### 1.4 Acceptance Criteria

**Phase 1 — Shortcuts + bridge → notes:**

- [ ] A documented Apple Shortcut invokes on a user-chosen phrase (e.g. “New
      Chronicle log entry”), runs `Dictate Text` with stop-after-pause, and
      HTTPS POSTs the transcript to the personal capture bridge.
- [ ] The bridge authenticates the request (bearer/token configured in the
      Shortcut) and appends a timestamped bullet to today’s personal notes file
      (`chronicles/notes/YYYY-MM-DD.md`) with a provenance tag (e.g. `[Siri]`).
- [ ] The Shortcut receives a success/failure ack suitable for Speak Text or a
      notification.
- [ ] Capture works off the LAN via a private tunnel (e.g. Tailscale) to an
      always-on host or the user’s machine — no public SaaS journal required.
- [ ] Setup is documented as a copy-paste recipe (no research project).

**Phase 2 — Activity envelope:**

- [ ] The same authenticated POST also writes a schema-valid
      `ActivityEnvelope` with source `ext:rosetta/siri-log` into the PRD-0015
      inbox (once inbox ingestion exists).
- [ ] The next synthesis run includes the voice log; submissions are idempotent
      on client-supplied `clientId`.

**Phase 3 — First-class App Intent:**

- [ ] Wayfinder (or a thin capture app) registers App Shortcut phrases so Siri
      invokes capture without a hand-built Shortcuts recipe; payload matches the
      Phase 1 bridge contract.

## 2. Users & Motivation

**Primary:** the engineer or knowledge worker who thinks in spoken journal
fragments while away from the keyboard — commute, kitchen, walk — and today
loses those thoughts or dumps them into a disposable Notes app.

**Secondary:** accessibility and hands-busy contexts that overlap PRD-0017 but
start from the OS, not from an already-open Wayfinder window.

Pain removed: “I said something important to Siri / myself and it never entered
the chronicle.”

## 3. Approach

Siri is a **producer**. Chronicle remains the ledger. The capture bridge is the
only new runtime surface; storage is PRD-0003 / PRD-0015.

| Layer     | Phase 1                                                  | Later                                                    |
| --------- | -------------------------------------------------------- | -------------------------------------------------------- |
| Client    | Apple Shortcuts (`Dictate Text` → `Get Contents of URL`) | App Intent / App Shortcut on Wayfinder (PRD-0010 / 0017) |
| Transport | HTTPS POST over private tunnel                           | Same contract                                            |
| Bridge    | Personal listener (HSR) on always-on host or workstation | Same                                                     |
| Landing   | Append daily notes (PRD-0003)                            | + inbox `ActivityEnvelope` (PRD-0015)                    |

```
User → Siri → Shortcuts (DictateText, stop after pause)
                → HTTPS POST + token → Capture bridge
                     → appendDaily (notes)
                     → inbox envelope (Phase 2)
                ← ack (speak / notify)
```

Implementation repo for the bridge is chosen at Accept (candidate homes:
`rosetta_chronicle` or a small companion under `rosetta_dev-scripts`). Classes
follow Handler → Service → Repository with InversifyJS.

## 4. Data Contracts

```ts
/** POST body from Shortcuts / App Intent to the personal capture bridge. */
export interface SiriChronicleLogRequest {
  /** Spoken journal text (already transcribed by Apple STT). */
  text: string;
  /** Client-side capture time (ISO-8601). */
  capturedAt: string;
  /** Idempotency key; bridge dedupes on (source, clientId). */
  clientId: string;
  source: "siri-shortcuts" | "app-intent";
}

export interface SiriChronicleLogResponse {
  ok: boolean;
  /** Path or date of the notes file updated, when applicable. */
  notesDate?: string;
  /** Inbox envelope id when Phase 2 writes activity. */
  activityId?: string;
  error?: string;
}
```

Auth: `Authorization: Bearer <token>` (or equivalent header) configured once in
the Shortcut; token stored only on the user’s devices and bridge host.

Phase 1 notes line shape (illustrative):

```md
- [HH:MM] [Siri] <transcript>
```

Phase 2 activity source: `ext:rosetta/siri-log`.

## 5. Constraints & Dependencies

- **Privacy / decentralization:** personal chronicle only; bridge is
  user-hosted ([ADR-0002](../architecture/ADR-0002-personal-vs-organizational-chronicle.md),
  [ADR-0005](../architecture/ADR-0005-decentralized-by-construction.md)).
- **Healthcare:** not a clinical documentation path; PHI must not be directed
  into shared or org chronicles.
- **Depends on** [PRD-0003](PRD-0003-notes-as-authoritative-input.md) (shipped)
  for notes append.
- **Soft-depends on** [PRD-0015](PRD-0015-chronicle-activity-schema-and-open-ingestion.md)
  Phase 2 inbox for activity envelopes.
- **Related:** [PRD-0016](PRD-0016-offline-on-device-intelligence.md) capture
  tier; [PRD-0017](PRD-0017-wayfinder-voice-audio-surface.md) in-app voice
  (complement, not substitute).
- **Architecture:** Handler / Service / Repository + InversifyJS for bridge
  TypeScript.

## 6. Risks & Open Questions

- **Siri phrase disambiguation:** Shortcuts phrases are user-defined; App
  Intents require `.applicationName` in at least one phrase — Phase 3 naming
  must stay speakable (“Chronicle”, “Wayfinder”, etc.).
- **Bridge availability:** laptop asleep ⇒ capture fails. Phase 1 docs should
  recommend an always-on host; queue-on-phone / retry is Future.
- **Apple STT quality / language:** OS-dependent; no Rosetta transcription in
  Phase 1 (optional on-device re-transcription later via PRD-0016).
- **Which repo owns the bridge?** Decide at Accept before Phase 1 spec.
- **Idempotency UX:** Shortcuts should generate a stable `clientId` per
  invocation (UUID action) so retries do not double-append.

## 7. Rollout & Phases

1. **Phase 1 — Shortcuts + bridge → notes:** documented Shortcut recipe,
   personal HTTPS bridge, authenticated append to today’s notes, ack path,
   private-tunnel setup.
2. **Phase 2 — Activity envelope:** same POST writes PRD-0015 inbox envelope
   (`ext:rosetta/siri-log`); synthesis picks it up; idempotent on `clientId`.
3. **Phase 3 — App Intent:** first-class Siri phrases via Wayfinder (or thin
   capture app); same bridge payload as Phase 1.

## 8. Future Considerations

- Apple Watch / CarPlay invocation of the same App Intent.
- On-device queue when the bridge is unreachable, flush on reconnect.
- Optional local re-transcription / cleanup through PRD-0016 before append.
- Remote authenticated ingest (PRD-0015 Future) if the personal bridge is
  replaced by a user-controlled endpoint with signed provenance — still not
  multi-tenant SaaS by default.
