---
id: PRD-0017
title: Wayfinder Voice & Audio Surface
status: Draft
date: 2026-07-31
owner: Russ Watson
related_adrs: [ADR-0001, ADR-0002, ADR-0003, ADR-0004, ADR-0005]
related_specs: []
supersedes: null
---

# PRD-0017: Wayfinder Voice & Audio Surface

> Everything Wayfinder can show, it can speak — and everything it can be asked
> by typing, it can be asked by voice. The knowledge layer becomes usable by
> people who cannot see a screen, and by everyone whose eyes and hands are
> busy.

## 1. Overview & Goals

### 1.1 Purpose

Wayfinder today is a visual surface: chronicles, notes, queue, and Ask
Wayfinder all assume a sighted reader at a keyboard. That excludes blind and
low-vision users entirely, underserves people who learn better by hearing, and
ignores every context where eyes are busy — commuting, walking, cooking,
driving.

The motivating scenario is accessibility-first: a blind user whose personal
chronicle contains a digitized copy of a physical book (captured by a
PRD-0015 physical-world producer — see §8 there) and who wants it **read
aloud**. Under the Marrakesh Treaty and 17 U.S.C. §121–121A, making and using
that accessible-format copy is lawful for a print-disabled person; Rosetta's
personal/org split (ADR-0002) keeps the copy private by construction, and the
promotion gate (PRD-0006/0009) is the compliance boundary that stops it from
ever leaving. What's missing is the consumption surface. That surface is this
PRD — and once it exists, it reads _everything_ in the chronicle, not just
books: today's chronicle on the morning walk, the queue while making coffee,
an artifact during a commute.

Voice input is the same boundary in the other direction: Ask Wayfinder by
speaking, capture a note by talking. PRD-0016 already defines `transcription`
as an inference task class; this PRD gives it a home in the app.

### 1.2 Goals

- **Read anything aloud:** every document Wayfinder renders — daily chronicles, notes, queue items, captured artifacts (including accessible-format book copies) — has a first-class "listen" affordance with play/pause, seek, speed, and position memory.
- **Voice in:** Ask Wayfinder accepts spoken questions; note capture accepts dictation. Both route through the PRD-0016 inference boundary (`transcription` task class), so speech never has to leave the device.
- **Screen-reader native:** the visual app meets WCAG 2.2 AA — semantic structure, focus management, full keyboard operability — so the audio surface complements rather than substitutes for assistive tech the user already runs.
- **Local-first speech:** TTS and STT default to on-device engines (OS-native voices or local neural TTS; local Whisper-class STT), consistent with PRD-0016's privacy postures and ADR-0005's no-required-server rule. Cloud voices are an opt-in quality upgrade, never a dependency.

### 1.3 Non-Goals

- **Not a robot or scanner project:** physical capture hardware is a PRD-0015 `ext:` producer — someone else's hardware speaking our schema.
- **Not an audiobook store or distribution channel:** accessible-format copies are personal, for the eligible owner's use; nothing here shares or syndicates content.
- **Not a general voice assistant:** the surface answers from the chronicle (evidence-first, ADR-0001), it does not do open-web tasks.

## 2. User Stories

1. As a **blind engineer**, I open Wayfinder with my screen reader, navigate to today's chronicle, and have it read aloud with controllable speed — and I ask follow-up questions by voice.
2. As a **print-disabled reader**, I listen to a book digitized into my personal chronicle by a capture rig, resuming exactly where I left off yesterday.
3. As an **auditory learner**, I replay the week's chronicles as a podcast-style queue during my commute.
4. As a **hands-busy user**, I dictate a note ("decided to move ingest validation into the repository layer") and it lands in today's notes file as authoritative input (PRD-0003).

## 3. Requirements

### 3.1 Audio out (TTS)

- A "listen" control on every readable view; document-to-speech converts the rendered Markdown structure (headings, lists, code fences announced sensibly, links spoken by label).
- Playback controls: play/pause, skip by section/paragraph, speed 0.5–3x, per-document resume position persisted locally.
- Engine selection follows the PRD-0016 backend descriptor pattern: OS-native TTS is the floor (always available offline); local neural TTS and cloud voices are optional upgrades chosen by configuration.

### 3.2 Voice in (STT)

- Push-to-talk and continuous dictation modes for Ask Wayfinder and note capture.
- Transcription routes through `IInferenceRepository` as the `transcription` task class (PRD-0016) — local-only posture means audio never leaves the device.
- Dictated notes append to the day's authoritative notes file exactly as typed notes do (PRD-0003); the chronicle records them identically.

### 3.3 Accessibility of the app itself

- WCAG 2.2 AA for the visual UI: semantic landmarks, heading hierarchy, focus order, visible focus, no keyboard traps, prefers-reduced-motion respected.
- All playback and dictation controls operable by keyboard and exposed to screen readers with correct roles/labels/state.
- Theme work (PRD-0014) inherits contrast requirements from this PRD.

### 3.4 Architecture

- Tauri thin-Rust-core rules apply (ADR-0003): audio device access in the Rust shell where required; business logic (document-to-speech segmentation, position tracking, routing) in TypeScript under HSR.
- Speech engines are repositories behind interfaces (`ITextToSpeechRepository`, transcription via PRD-0016's inference boundary) — swappable per device and posture, like every other backend.
- Everything works with no server (ADR-0005 design test): OS voices + local STT give the full loop offline.

## 4. Success Metrics

- A blind user can complete the core loop — open, navigate, listen to a chronicle, ask a question by voice, hear the answer — with no sighted assistance.
- Listen-to-first-audio latency under 1s for OS-native voices on a typical document.
- 100% of readable views expose the listen affordance; audit passes WCAG 2.2 AA.

## 5. Rollout & Phases

1. **Phase 1 — Read the chronicle aloud:** OS-native TTS, listen control on chronicle/notes/queue views, playback controls + resume, WCAG 2.2 AA audit of existing UI.
2. **Phase 2 — Voice in:** push-to-talk Ask Wayfinder and note dictation through the PRD-0016 transcription task class, local-first.
3. **Phase 3 — Long-form listening:** artifact/book-length documents — chapter navigation, sleep timer, podcast-style queue of recent chronicles, optional neural TTS upgrade.

## 6. Open Questions

- Voice output for Ask Wayfinder answers: stream TTS as tokens arrive, or wait for complete sections? (Streaming feels right; needs prosody experiments.)
- Does long-form listening belong in Wayfinder proper or a companion mobile surface (which PRD-0016's capture tier already anticipates)?
- Wake-word/continuous listening: deliberately out of scope until the privacy story is airtight — revisit after Phase 2.

## 7. Future Considerations

- **Audio as capture:** meetings and voice memos transcribed into activity envelopes (PRD-0015) — the surface's STT machinery reused as a producer.
- **Voices with provenance:** if generated narration is ever shared inside an org, it carries `Model-Provenance` metadata like any synthesized artifact (ADR-0007 trailers, PRD-0016 sidecar).
- **Braille display support:** beyond audio — chronicle content structured for refreshable braille via the same semantic document model.
