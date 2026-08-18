# architecture

Architecture decisions and history for the Rosetta platform.

This folder holds Architecture Decision Records (ADRs), system diagrams, and the evolving design of
how Chronicle, Wayfinder, and future products fit together. Over time, Chronicle itself will help
capture and surface this architecture history — but human-authored ADRs live here.

[`CONCEPTUAL-MODEL.md`](CONCEPTUAL-MODEL.md) is the architect-altitude sketch of
*what the system is*. It is not an ADR. Do not collapse it into daily-chronicle
Activity, and do not put its types into the essay.

All TypeScript in Rosetta follows the Handler / Service / Repository + InversifyJS pattern
(see `../.claude/rules/architecture-hsr.md`); record deviations or extensions to that pattern as ADRs here.
