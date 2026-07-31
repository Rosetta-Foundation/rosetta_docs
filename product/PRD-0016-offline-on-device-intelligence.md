---
id: PRD-0016
title: Offline On-Device Intelligence
status: Draft
date: 2026-07-31
owner: Russ Watson
related_adrs: [ADR-0001, ADR-0002, ADR-0003, ADR-0005]
related_specs: []
supersedes: null
---

# PRD-0016: Offline On-Device Intelligence

> Chronicle and Wayfinder run their full knowledge loop offline, on local
> agentic models, on hardware down to phones and wearables — capture anywhere,
> synthesize wherever compute lives, sync when connected.

## 1. Overview & Goals

### 1.1 Purpose

Rosetta's storage layer is already offline-first — git works on a plane. Its
intelligence layer is not: synthesis, distillation, tagging, and Wayfinder
queries all route to the Claude API (PRD-0010 codified this: "the intelligence
is Claude; Wayfinder is the delivery mechanism"). Lose connectivity and Rosetta
degrades from a knowledge platform to a filing cabinet.

That coupling also caps where Rosetta can live. The knowledge worth capturing
doesn't stop at the desk — decisions happen in hallways, ideas on walks,
incidents at 2am from a phone. On-device inference crossed the practicality
threshold in 2025–2026: quantized 1–4B models run capably on phone NPUs, and
sub-billion models handle summarization and classification. The intelligence
layer should be a pluggable boundary — cloud model, local model, or some future
runtime — chosen per device, per task, per privacy posture. Model-agnostic is
the same move as API-first (PRD-0015), applied to inference instead of
ingestion.

### 1.2 Goals

- Make inference a swappable boundary: the same synthesis service runs against the Claude API, a local runtime (llama.cpp/GGUF, MLX, ExecuTorch), or any future backend — selected by configuration, not code changes.
- Run the complete daily loop — capture, synthesis, tagging, persistence — fully offline on a desktop with a local model.
- Define capability tiers so constrained devices participate at the level their hardware allows, down to capture-only wearables.
- Decouple capture from synthesis: a device that can't synthesize queues schema-valid activity (PRD-0015) for deferred synthesis wherever compute exists.
- Strengthen the ADR-0002 privacy story: a personal chronicle whose intelligence runs on-device never sends personal content to any third party.

### 1.3 Non-Goals

- Not training or fine-tuning a custom model — Rosetta consumes models, it does not produce them.
- Not abandoning cloud models — hybrid is the steady state; frontier models remain the quality ceiling for heavy reasoning when online and permitted.
- Not building wearable hardware or a watch app in the near term — this PRD establishes the architecture that makes one possible.
- Not promising cloud-parity output quality from local models — quality tiers are explicit and surfaced, not hidden.
- Not a general local-AI runtime for arbitrary tasks — scoped to Rosetta's knowledge loop (synthesis, distillation, tagging, query).

### 1.4 Acceptance Criteria

**Phase 1 — Inference abstraction:**

- [ ] All LLM calls in Chronicle and Wayfinder go through a single `IInferenceRepository` boundary; no service constructs a provider client directly.
- [ ] Backends are selected by configuration (per machine, per task class) with no code changes.
- [ ] A `ModelCapabilityDescriptor` describes each backend (context window, task classes supported, quality tier) and services degrade gracefully against it.

**Phase 2 — Offline desktop loop:**

- [ ] With networking disabled, a local backend (e.g. llama.cpp or MLX with a 4–12B quantized model) produces a complete daily chronicle: synthesis, session distillation, tag inference, sidecar persistence.
- [ ] Output produced offline is marked with its model provenance (backend, model, quality tier) in the sidecar, so a later cloud-powered regeneration can improve it deliberately (respecting the PRD-0005 clobber guard).
- [ ] Wayfinder's "Ask" answers from local RAG + local model when offline, with cited evidence, and clearly indicates degraded mode.

**Phase 3 — Hybrid routing:**

- [ ] A routing policy assigns task classes to backends: routine tasks (titles, tags, short summaries) run local-first; heavy reasoning prefers cloud when available.
- [ ] The user can pin the privacy posture: `local-only`, `local-preferred`, or `cloud-preferred` — enforced at the inference boundary, per ADR-0002's owner-controls-the-data principle.

**Phase 4 — Constrained-device capture:**

- [ ] A capture-only reference client (CLI simulating a wearable) emits schema-valid `ActivityEnvelope`s (PRD-0015) while fully offline and syncs them to the inbox when connectivity returns.
- [ ] Deferred synthesis picks up synced envelopes and folds them into the correct day's chronicle — late-arriving activity regenerates the affected window.

## 2. Users & Motivation

**Primary:** the engineer whose day doesn't stop when connectivity does —
planes, trains, secure facilities, bad hotel wifi. Their chronicle should not
have gaps that correlate with their network.

**Privacy-sensitive users and orgs:** ADR-0002 promises the personal chronicle
is private, but today its _processing_ transits a third-party API. Local-only
inference closes that gap completely — a meaningful unlock for regulated
environments.

**Future ambient capture:** the person who has their best architectural idea on
a run. A watch that records "note to self: the retry storm is a backpressure
problem, not a timeout problem" as a schema-valid activity — synthesized into
the chronicle that evening — is the logical endpoint of "knowledge should
accumulate automatically" (ADR-0001).

Pain removed:

- **No connectivity-shaped holes in the ledger.**
- **No forced privacy trade-off** between using intelligence and keeping personal content on-device.
- **No "I'll write it down later"** — capture moves to where the moment happens.

## 3. Approach

### 3.1 The inference boundary

```
ChronicleService / KnowledgeService / StandupService
        │  InferenceRequest { taskClass, prompt, constraints }
        ▼
┌───────────────────────────────┐
│ IInferenceRepository          │   routing policy + privacy posture
│   ├─ ClaudeApiBackend         │   frontier quality, online only
│   ├─ LocalRuntimeBackend      │   llama.cpp / GGUF, MLX (Apple Silicon),
│   │                           │   ExecuTorch (mobile) — offline capable
│   └─ (future backends)        │   NPU runtimes, whatever comes next
└───────────────────────────────┘
```

Services never know which backend answered; they know the declared quality
tier and adapt (e.g. simpler prompts, shorter outputs, `reviewNeeded` marking
for low-tier distillations — the marker and pathway already exist).

### 3.2 Capability tiers

| Tier        | Hardware                                  | What runs locally                                                                     | What defers                          |
| ----------- | ----------------------------------------- | ------------------------------------------------------------------------------------- | ------------------------------------ |
| **Full**    | Desktop/laptop (≥16GB, GPU/Apple Silicon) | Entire loop: synthesis, distillation, tags, RAG query (4–12B quantized)               | Nothing (cloud optional for quality) |
| **Light**   | Phone/tablet (NPU, 1–4B models)           | Capture, note distillation, short summaries, simple queries                           | Full daily synthesis                 |
| **Capture** | Wearable/embedded (sub-1B or no model)    | Capture only: voice-note transcription at best, schema-valid activity emission always | All synthesis and query              |

The load-bearing insight: **the Capture tier requires no model at all.**
PRD-0015's envelope + inbox protocol is the entire integration surface — a
watch is just an activity producer with a clock and a sync path. Intelligence
concentrates where compute lives; capture spreads to where life happens.

### 3.3 Deferred synthesis

Constrained devices append envelopes to a local queue; on sync, envelopes land
in the inbox (PRD-0015 Phase 2) and a regeneration folds them into the right
day. Late-arriving activity is already a solved shape — it is exactly the
regeneration path PRD-0002/PRD-0005 built, triggered by ingestion instead of
by the user.

### 3.4 Hybrid routing (steady state)

Local models are genuinely good at Chronicle's routine work — summarization
and classification are exactly what sub-4B models do well. Frontier models
stay meaningfully better at long-context synthesis and nuanced reasoning. So:
route by task class, respect the privacy posture, and record model provenance
so any output can be regenerated at a higher tier later. Quality is a
property of the artifact, not a global mode.

## 4. Data Contracts

```ts
export type InferenceTaskClass =
  | "synthesis" // full daily chronicle composition
  | "distillation" // session/note title + summary
  | "tagging" // taxonomy inference
  | "query" // RAG-grounded Q&A
  | "transcription"; // voice note → text (capture tier)

export type QualityTier = "frontier" | "capable" | "minimal";

export interface ModelCapabilityDescriptor {
  backendId: string;
  /** e.g. "claude-api", "llama-cpp", "mlx", "executorch". */
  runtime: string;
  model: string;
  contextWindowTokens: number;
  supportedTasks: InferenceTaskClass[];
  qualityTier: QualityTier;
  offlineCapable: boolean;
}

export type PrivacyPosture =
  "local-only" | "local-preferred" | "cloud-preferred";

/** Stamped into the sidecar for every generated artifact. */
export interface ModelProvenance {
  backendId: string;
  model: string;
  qualityTier: QualityTier;
  generatedAt: string;
}

/** Queue entry on a capture-tier device awaiting sync + synthesis. */
export interface DeferredSynthesisTask {
  /** The captured activity, already schema-valid (PRD-0015). */
  envelope: ActivityEnvelope;
  capturedAt: string;
  syncedAt?: string;
}
```

## 5. Constraints & Dependencies

- **Depends on PRD-0015:** the schema + envelope + inbox protocol is what makes capture-tier devices possible and what carries deferred activity.
- **Architecture:** HSR + InversifyJS — the inference boundary is a Repository; routing policy is Service logic; backends are lower-level repositories.
- **Physics:** on-device decode is memory-bandwidth bound (mobile ~50–90 GB/s vs. data-center 2–3 TB/s). Budget for 4-bit quantization, small contexts, and chunked synthesis on Light-tier devices — don't design prompts that assume frontier context windows everywhere.
- **Runtime landscape (2026):** llama.cpp/GGUF for desktop CPU + prototyping, MLX for Apple Silicon, ExecuTorch for mobile production. Pin per-platform; the boundary exists precisely so these can churn.
- **Privacy (ADR-0002):** `local-only` posture must be enforceable and auditable at the boundary — it is a promise, not a preference.
- **Tauri/ADR-0003:** Wayfinder's thin-Rust-core stance fits a sidecar local-inference process; model weights are a per-machine asset, never committed to any repo.

## 6. Risks & Open Questions

- **Quality gap:** local synthesis will read worse than Claude's. Mitigation: provenance stamping + deliberate cloud regeneration; `reviewNeeded` surfacing; keep expectations explicit in the UI.
- **Model distribution:** who downloads/updates weights, and from where? Likely a first-run fetch with checksums; needs a story for air-gapped machines (bundled model option).
- **Battery and thermals on mobile:** background synthesis on a phone may be unacceptable; Light tier may need to run distillation only while charging. Profile on real hardware early.
- **Wearable transcription:** is on-watch speech-to-text good enough, or does audio sync to the phone for transcription? (Leaning: transcribe at the Light tier, watch emits audio-note envelopes.)
- **Regeneration churn:** late-arriving envelopes trigger regenerations; a burst of synced activity could regenerate many days. Batch by window before regenerating.
- **Licensing:** local model licenses (Llama, Gemma, Qwen, Phi) differ on commercial terms — pick defaults with clean licensing and make the model slot user-swappable.

## 7. Rollout & Phases

1. **Phase 1 — Inference abstraction:** introduce `IInferenceRepository` + capability descriptors; migrate all existing Claude calls behind it; configuration-driven backend selection.
2. **Phase 2 — Offline desktop loop:** local runtime backend (llama.cpp and/or MLX); full offline daily chronicle; model provenance in the sidecar; offline Wayfinder Ask in degraded mode.
3. **Phase 3 — Hybrid routing:** task-class routing policy; privacy postures enforced at the boundary; deliberate quality-upgrade regeneration.
4. **Phase 4 — Constrained-device capture:** deferred-synthesis queue; capture-only reference client proving the wearable shape end-to-end against the PRD-0015 inbox.

## 8. Future Considerations

- **A real wearable client:** watchOS/Wear OS capture app — voice notes, quick tags, queue-item check-offs — emitting envelopes over the phone sync path.
- **On-device personalization:** local fine-tuning/adapters so the personal chronicle's model learns the owner's vocabulary and projects without any data leaving the device.
- **Test-time compute for small models:** small-model quality on hard tasks improves markedly with search/self-consistency strategies — a lever for closing the quality gap offline.
- **Peer synthesis:** a phone offloading synthesis to the owner's desktop over the local network (still zero-cloud) — compute follows trust, not the internet.
- **Org-side local inference:** running the coherence protocol (PRD-0009) checks on a self-hosted model for orgs that cannot send org knowledge to external APIs.
