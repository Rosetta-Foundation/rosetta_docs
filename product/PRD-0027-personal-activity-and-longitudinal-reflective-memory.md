---
id: PRD-0027
title: Personal Activity & Longitudinal Reflective Memory
status: Draft
date: 2026-08-16
owner: Russ Watson
related_adrs: [ADR-0002, ADR-0005]
related_specs: []
supersedes: null
---

# PRD-0027: Personal Activity & Longitudinal Reflective Memory

> Extend Chronicle beyond engineering activity so a person can preserve private
> experience, expression, reflection, correction, and choice as a traceable
> trajectory without making AI an authority over their identity.

## 1. Overview & Goals

### 1.1 Purpose

Chronicle records what happened in engineering, but its current activity
contract and daily rendering still treat engineering as the only domain. A
personal history such as a ChatGPT export contains a different kind of durable
context: not only conclusions, but the conversations, uncertainty,
contradictions, rejected interpretations, and revisions through which a person
changed their mind. Flattening that history into session titles would lose the
reasoning path that makes it valuable.

This capability introduces personal activity and a reflective layer over one
neutral Chronicle engine. Source evidence remains immutable and attributable;
AI reflections remain proposals; and human recognition, rejection, and
correction become first-class parts of the record. The resulting system
preserves the trajectory of becoming while keeping self-authorship with the
person.

### 1.2 Goals

- Represent personal activity distinctly from engineering activity without
  forking Chronicle or creating a second engine.
- Import a ChatGPT history as the first proving source while preserving its
  conversation graph, message provenance, and vendor-neutral normalized form.
- Preserve raw source evidence separately from derived summaries,
  interpretations, and patterns.
- Record AI-produced reflections as non-authoritative proposals that the person
  can recognize, reject, correct, or leave uncertain.
- Preserve superseded and rejected thought without presenting it as current
  truth.
- Make longitudinal trajectories queryable across expression, challenge,
  revision, decision, and outcome.
- Keep personal material private by default and prevent any automatic path into
  a shared chronicle.
- Produce human-readable views and machine-readable records from the same
  durable source of truth.

### 1.3 Non-Goals

- Not claiming that Chronicle or an AI model "knows" the person or owns a
  canonical model of their identity.
- Not treating generated reflections, inferred emotions, or detected patterns
  as facts without explicit human evaluation.
- Not importing personal history into organizational or shared chronicles.
- Not building a clinical, diagnostic, therapeutic, or crisis-intervention
  system.
- Not inferring intoxication, mental state, or other sensitive state labels from
  text; state context may be recorded only when the person explicitly supplies
  it.
- Not designing the normalized contract around undocumented details of one
  ChatGPT export before a real export has been inventoried.
- Not making every source message a polished journal entry or discarding messy,
  contradictory, branched, or superseded source material.
- Not implementing cross-person, relationship, team, organization, or civic
  reflective memory in the initial phases.
- Not defining promotion of reflective records; existing intentional promotion
  boundaries remain in force.

### 1.4 Acceptance Criteria

**Phase 1 — Export inventory and evidence-backed contract:**

- [ ] A read-only inventory command accepts an OpenAI ChatGPT export directory
      or archive and reports conversation count, message/node count, date range,
      roles, content types, attachment references, branch topology, and
      unsupported records without writing to a chronicle.
- [ ] The inventory produces no source-content output by default beyond the
      minimum metadata needed to review structure and privacy exposure.
- [ ] A sanitized fixture derived from the observed export structure covers
      branches, missing timestamps, non-text content, attachments, malformed
      nodes, and deleted or unavailable messages found in the real export.
- [ ] The proposed normalized conversation contract is revised against the
      inventory findings before import implementation begins.

**Phase 2 — Private, idempotent personal-history import:**

- [ ] Importing the same export more than once produces no duplicate archive,
      conversation, node, activity, or evidence records.
- [ ] The importer preserves stable source identifiers, parent/child topology,
      selected-current-node information when available, message role,
      timestamps, content types, model/producer provenance when available, and
      attachment references.
- [ ] Conversations are cataloged as artifacts and projected into
      `personal`-domain activity without appearing in engineering-only daily
      summaries.
- [ ] Normalized records retain exact evidence references to source archive,
      conversation, and node identifiers.
- [ ] Raw export content is never committed unencrypted to a Git repository;
      the selected source-vault strategy is documented, portable, and explicit
      about key ownership and recovery.
- [ ] Import targets only the owner's personal chronicle and creates no
      promotion, synchronization, or shared-chronicle side effect.
- [ ] Unsupported content is reported and preserved by reference rather than
      silently discarded or partially interpreted.

**Phase 3 — Human-authored correction loop:**

- [ ] A reflection can cite one or more evidence or reflective records and
      classify itself as an observation, emotion, interpretation, hypothesis,
      value, decision, outcome, revision, or pattern.
- [ ] Every reflection identifies its producer and distinguishes human
      expression from model-generated suggestion.
- [ ] A model-generated reflection remains `suggested` until the person records
      a `recognized`, `rejected`, `corrected`, or `uncertain` evaluation.
- [ ] Evaluations are append-only records; correcting a reflection does not
      rewrite or delete the reflection or an earlier evaluation.
- [ ] A person can complete and inspect one end-to-end chain from source
      expression through suggested reflection to recognition or rejection and a
      subsequent revision or choice.
- [ ] Views clearly distinguish source evidence, generated reflection, human
      evaluation, and computed current understanding.

**Phase 4 — Longitudinal trajectories:**

- [ ] Records can be linked with `supports`, `challenges`, `revises`, `rejects`,
      `led_to`, `resulted_in`, and `reflects_on` relationships.
- [ ] A trajectory query can return what was expressed, what challenged it,
      which reflection was accepted or rejected, what changed, and what
      decision or outcome followed, with provenance for every step.
- [ ] Earlier and rejected records remain discoverable but are not presented as
      current beliefs without their later evaluations and revisions.
- [ ] A personal timeline can render personal activity independently or
      alongside engineering activity without conflating their domains.
- [ ] Pattern generation cites the records from which the pattern was derived
      and enters the same human evaluation loop as every other generated
      reflection.

## 2. Users & Motivation

The primary user is a person who has accumulated meaningful thinking in
journals, AI conversations, notes, and future capture surfaces, but cannot see
how their interpretations, values, and choices evolved. A final conclusion can
show what they believe now; the reasoning trail shows how they became capable
of believing it and where they may still be wrong.

The first proving case is Russ's ChatGPT history export. It contains personal
activity that is distinct from engineering activity and may include branched
conversations, incomplete ideas, emotional reactions, third-party information,
and model-generated interpretations. The user needs to catalog it without
prematurely treating every message as truth or exposing private material.

Future agents and Wayfinder benefit from explicit provenance and epistemic
boundaries. They can retrieve prior reasoning and surface contradictions
without silently converting an old statement or an AI suggestion into the
person's present identity.

## 3. Approach

The capability separates capture, reflection, and presentation:

```text
source archive
    ↓
normalized conversation graph
    ↓
personal-domain activity + evidence
    ↓
human expression / suggested reflection
    ↓
recognition / rejection / correction / uncertainty
    ↓
trajectory links
    ↓
computed current understanding
```

### 3.1 Orthogonal dimensions

`domain` answers where activity belongs. `kind` answers what a reflective
statement represents. `disposition` answers how the person has evaluated a
suggestion. These dimensions must not be collapsed:

| Dimension   | Examples                                                        |
| ----------- | --------------------------------------------------------------- |
| Domain      | `personal`, `engineering`, future namespaced domains            |
| Kind        | observation, emotion, hypothesis, value, decision, revision     |
| Producer    | human, imported assistant, Chronicle inference backend          |
| Disposition | suggested, recognized, rejected, corrected, uncertain           |

An AI-produced `pattern` in the `personal` domain is still only `suggested`
until the person evaluates it.

### 3.2 Two-layer import

The import path preserves both:

1. A source archive whose bytes and content hash establish immutable
   provenance. Raw sensitive content uses a portable source-vault strategy and
   is not committed unencrypted to Git.
2. A vendor-neutral conversation graph used by Chronicle. It preserves
   topology rather than flattening ChatGPT's node mapping into one transcript.

The importer first supports inventory-only operation. Durable import is a
separate, explicit action after the user reviews the inventory and storage
policy.

### 3.3 Append-only reflection

Source evidence and prior reflections are not rewritten when understanding
changes. A human evaluation cites the reflection it evaluates. A correction
adds the corrected account and relationship edges; it does not erase the
misunderstanding. "Current understanding" is therefore a materialized view over
history, never a mutable identity profile.

### 3.4 Neutral engine, policy-bearing repository

Chronicle applies the same capture and evidence machinery to all domains. The
personal repository supplies the privacy and sharing policy. Domain-specific
rendering may differ, but personal and engineering records use common
provenance, storage, and query primitives.

## 4. Data Contracts

The exact source metadata fields remain provisional until Phase 1 inventories a
real export. The semantic boundaries are stable:

```ts
export type ActivityDomain =
  | 'personal'
  | 'engineering'
  | `ext:${string}`;

export interface SourceArchive {
  id: string;
  source: `ext:${string}/${string}`;
  contentHash: string;
  importedAt: string;
  storageRef: string;
  formatVersion?: string;
}

export interface ConversationArtifact {
  id: string;
  archiveId: string;
  sourceId: string;
  title?: string;
  createdAt?: string;
  updatedAt?: string;
  currentNodeId?: string;
  nodeIds: string[];
}

export type ConversationRole =
  | 'human'
  | 'assistant'
  | 'system'
  | 'tool'
  | 'unknown';

export interface ConversationNode {
  id: string;
  conversationId: string;
  sourceId: string;
  parentId?: string;
  childIds: string[];
  role: ConversationRole;
  createdAt?: string;
  contentType: string;
  contentRef: string;
  producer?: {
    name: string;
    model?: string;
  };
}

export type ReflectiveKind =
  | 'observation'
  | 'emotion'
  | 'interpretation'
  | 'hypothesis'
  | 'value'
  | 'decision'
  | 'outcome'
  | 'revision'
  | 'pattern';

export interface ReflectiveRecord {
  id: string;
  domain: ActivityDomain;
  kind: ReflectiveKind;
  text: string;
  evidenceRefs: string[];
  producedAt: string;
  producer: {
    type: 'human' | 'agent' | 'imported-assistant';
    name: string;
    model?: string;
  };
}

export type ReflectionDisposition =
  | 'recognized'
  | 'rejected'
  | 'corrected'
  | 'uncertain';

export interface ReflectionEvaluation {
  id: string;
  reflectionId: string;
  disposition: ReflectionDisposition;
  evaluatedAt: string;
  evaluator: 'human';
  correctionRef?: string;
  rationale?: string;
  supersedesEvaluationId?: string;
}

export type TrajectoryRelation =
  | 'supports'
  | 'challenges'
  | 'revises'
  | 'rejects'
  | 'led_to'
  | 'resulted_in'
  | 'reflects_on';

export interface TrajectoryLink {
  id: string;
  fromRef: string;
  toRef: string;
  relation: TrajectoryRelation;
  createdAt: string;
  producer: 'human' | 'agent';
}
```

Source archive bytes are evidence, not the normalized query contract.
Normalized content references may point into the encrypted source vault or to
owner-approved extracted content. The Phase 1 inventory determines which
ChatGPT-specific metadata belongs in an extension field rather than the common
contract.

## 5. Constraints & Dependencies

- **Human agency:** reflection is not authority. Only the person can recognize,
  reject, or correct a proposed reflection about themselves.
- **Privacy:** personal history is private by default under ADR-0002. Raw
  exports may include secrets, health information, intimate content, and
  third-party information; private Git hosting alone is not a sufficient
  deletion or secret-management strategy.
- **Decentralization:** storage and querying must retain core capability without
  a required hosted service under ADR-0005.
- **Evidence first:** every imported activity, reflection, evaluation, and
  trajectory link must retain attributable provenance.
- **Neutral engine:** domain separation is data and policy, not a fork or
  personal-mode branch in Chronicle.
- **Architecture:** implementation uses Handler → Service → Repository with
  InversifyJS. Import parsing and source-vault access are repository concerns;
  normalization and reflection policy are service concerns.
- **Open ingestion:** PRD-0015 provides the future schema-first ingestion
  boundary. This PRD may extend its activity contract with domain metadata but
  does not duplicate its transport.
- **Promotion:** ADR-0002 and PRD-0006 remain the only intentional path from a
  personal chronicle toward shared knowledge. This capability adds no automatic
  promotion.
- **Real-export dependency:** Phase 1 requires an owner-provided ChatGPT export
  before the normalized import contract is finalized.

## 6. Risks & Open Questions

- **Source-vault design:** choose between local encrypted files, encrypted
  content-addressed objects, or another portable mechanism after measuring the
  export and attachment shapes. Key loss, backup, rotation, and selective
  deletion must be addressed before import.
- **Git durability versus deletion:** append-only epistemic history is valuable,
  but personal-data deletion must remain possible. Raw evidence retention and
  normalized ledger retention may require different policies.
- **Third-party privacy:** conversations can contain information about people
  who did not consent to durable analysis. The product needs owner-controlled
  exclusion and redaction before any derived reflection is promoted or shared.
- **Inference overreach:** longitudinal pattern detection can create persuasive
  but false narratives. Provenance, uncertainty, and the evaluation loop reduce
  this risk but do not eliminate it.
- **Classification ambiguity:** one statement may be both emotion and
  interpretation, or a decision may later become evidence of a value. Decide
  after proving whether records need one primary kind plus secondary labels.
- **Conversation boundaries:** ChatGPT titles and conversation containers may
  not match meaningful reflective episodes. Preserve source boundaries first;
  derive episodes later without destroying topology.
- **Current-understanding view:** conflicting recognized evaluations may coexist.
  The view must surface unresolved conflict rather than manufacture consensus.
- **Attachment coverage:** export archives may reference files that are absent,
  transformed, or encoded separately. Inventory must report completeness.
- **Sensitive-state continuity:** user-supplied context such as "high" or
  "sober" can be useful, but the system must not infer these labels or appoint
  one state as the authoritative self.

## 7. Rollout & Phases

1. **Phase 1** — Build the read-only ChatGPT export inventory, inspect a real
   owner-provided export, create sanitized fixtures, and finalize the
   evidence-backed normalized conversation contract.
2. **Phase 2** — Implement explicit, idempotent import into a private source
   vault, conversation graph, artifacts, evidence, and personal-domain activity
   while preserving engineering-domain separation.
3. **Phase 3** — Add reflective records and the append-only human recognition,
   rejection, correction, and uncertainty loop; prove one complete reflective
   chain.
4. **Phase 4** — Add trajectory relationships, current-understanding views,
   personal timeline rendering, and provenance-backed longitudinal queries and
   pattern suggestions.

## 8. Future Considerations

- Additional personal sources: journals, voice logs from PRD-0018, messages,
  photos, calendar context, health exports, and other owner-controlled archives.
- Selective forgetting, retention windows, cryptographic erasure, and
  re-import-safe deletion markers.
- Local inference from PRD-0016 for sensitive reflective material that should
  not leave the device.
- Wayfinder interfaces for reviewing suggested reflections, traversing belief
  revisions, and comparing explicit user-supplied cognitive or emotional
  contexts.
- Reflection across relationship, team, organization, community, and civic
  scales, with consent and policy appropriate to each scale.
- Promotion of owner-selected, sanitized insights whose shared evidence stands
  independently of the private reflective trail.
- Temporal queries such as "what did I believe before this decision?", "which
  rejected hypotheses later became relevant?", and "what changed after I acted?"
- A general principle for Rosetta at every scale: preserve why a system became
  what it is while allowing it to become something else.
