---
id: PRD-0027-ephemeral-store-loss-2026-08
title: Ephemeral private store loss — 2026-08
date: 2026-08-23
prd: PRD-0027
status: Working record
source: Operator process note after accepted private-artifact loss
---

# Ephemeral private store loss — 2026-08

Sanitized **process** note only. It is not an engine-semantics claim, not
an architecture change, not E8, and not a reconstruction of lost
artifacts.

Type: **H** (process). Confidence: observed host fact, not a product
hypothesis.

## Record

- Private experimental artifacts were stored only in ephemeral
  filesystem space (`/tmp`).
- After a host reboot, that store was gone.
- Sanitized research checkpoints in this repository survived.
- The original private artifacts did not.
- Future longitudinal specimens require durable private storage, not
  `/tmp` alone.

Do not reconstruct the lost store from chat, docs, or memory.
Sanitized checkpoints are not a substitute for the original artifacts.
