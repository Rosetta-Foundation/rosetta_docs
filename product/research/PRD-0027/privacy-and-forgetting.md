---
id: PRD-0027-privacy-and-forgetting
title: Privacy, correction, and forgetting gate
date: 2026-08-16
prd: PRD-0027
---

# Privacy, correction, and forgetting

**Gate:** no persistent-capture prototype (ChatGPT import into a live
chronicle, always-on logging, or model-written biography) until this file is
reviewed and the unchecked items below are either done or explicitly deferred
with a reason.

This is central design material (Feedback 3), not a later compliance pass.

## Floor already in PRD-0027

Do not relitigate these. Research may add tests; it may not weaken them.

- Personal material is private by default ([ADR-0002](../../../architecture/ADR-0002-personal-vs-organizational-chronicle.md)).
- No automatic promotion or shared-chronicle side effect.
- Raw export content is not committed unencrypted to Git.
- Reflections are proposals until the person evaluates them.
- Evaluations are append-only; correction does not rewrite the past.
- The system does not claim to know the person or infer sensitive states.
- Forgetting and exclusion are part of the practice, not a defect.
- Private Git hosting is not a sufficient deletion or secret-management story.

## Controls that must be thinkable before capture

A person must be able to predict the effect of each control.

| Control | Meaning | Predictable effect |
| ------- | ------- | ------------------ |
| **Withhold** | Never ingest this source, conversation, or span | It does not enter the vault or the ledger |
| **Redact** | Keep the waypoint, remove or replace content | Downstream reflections cannot see the redacted text |
| **Correct** | Add an evaluation or revised account | Old record remains; current-understanding view updates |
| **Suppress** | Hide from default views and inference | Still recoverable by the owner until deleted |
| **Expire** | Time-bounded retention | After expiry, inference and default views cannot use it |
| **Delete** | Remove from vault and ledger per policy | Re-import must not silently resurrect it without notice |
| **Bound** | Mark private / shared / dormant | Sharing never happens by adjacency or “the graph connected it” |

If a prototype cannot explain these in one sitting, it is not ready.

## What must remain unrecorded

A path system stays humane only if some things are allowed to fade.

- Unselected everyday activity (clicks, location, ambient audio)
- Inferred intoxication, diagnosis, or other sensitive state labels
- Third-party content the owner did not choose to keep
- Model-generated identity claims (“this is who you are”)
- Anything the owner would not put in a private letter they might later burn

## Review checklist (before capture)

- [ ] Owner can withhold at import time (conversation, date range, attachment)
- [ ] Inventory mode exists and writes nothing (PRD-0027 Phase 1)
- [ ] Deletion and re-import interaction is specified (tombstone or equivalent)
- [ ] Git history cannot undelete raw personal content the owner erased
- [ ] Current-understanding views do not treat suppressed or rejected
      reflections as present beliefs
- [ ] No engagement metric depends on volume captured or paths created
- [ ] Failure-mode table started (changed beliefs, false provenance, coercive
      sharing, emotionally loaded memories)
- [ ] Experiment 4, if run, uses a disposable mock — not a live ledger

## Research questions (do not block the gate on answers)

- Can people predict suppress vs delete vs expire?
- Does offering forgetting reduce or increase capture anxiety?
- When does append-only history conflict with a deletion request?

Those belong in Experiment 4 after this gate is reviewed.
