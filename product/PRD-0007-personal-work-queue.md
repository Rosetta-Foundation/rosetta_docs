---
id: PRD-0007
title: Personal Work Queue
status: Accepted
date: 2026-07-24
owner: Russ Watson
related_adrs: [ADR-0002]
related_specs: []
supersedes: null
---

# PRD-0007: Personal Work Queue

> A unified, Chronicle-native intake queue that answers "what's next?" — replacing
> fragmented Jira views, notepads, and mental state with a single structured queue
> that feeds from every source and closes automatically as work is recorded.

## 1. Overview & Goals

### 1.1 Purpose

Chronicle captures the output of work. But the input side — deciding *what to do
next* — is fragmented across Jira boards, handwritten notepads, mental state, and
scattered TODO comments in session transcripts. The engineer re-derives their
working context at the start of every session: "where was I, what was I doing,
what should I do now?"

This is the same problem Chronicle solves for the past ("what did I do?"),
applied to the future ("what should I do next?"). A personal work queue unifies
every source of incoming work into one prioritized view the engineer owns — not
a team board optimized for managers, but a personal queue optimized for flow.

Jira remains the organizational source of truth (planning, velocity, story
points, cross-team visibility). The personal queue **mirrors** Jira assignments
as one input source among many — alongside PRD phases, session follow-ups,
ad-hoc ideas, and Slack requests. It is a *consumption view*, not a replacement
for the team's planning system.

The automation endgame: when a feature request, bug report, or Jira assignment
arrives, an agent triages it into the queue, links context, suggests priority,
and when the engineer asks "what's next?" — it answers from structured data, not
from memory. When Chronicle records matching activity, the loop closes
automatically.

### 1.2 Goals

- Provide a single structured queue that answers "what should I do next?" from one place.
- Pull work items from every source: Jira assignments, unbuilt PRD phases, session follow-ups, ad-hoc ideas, Slack requests.
- Close the loop with Chronicle: completing work recorded in the daily chronicle automatically resolves the corresponding queue item.
- Keep the queue machine-readable so an agent can triage, prioritize, and suggest "next" without the engineer re-deriving context.
- Preserve human agency: the queue suggests and orders; the engineer decides.

### 1.3 Non-Goals

- Not replacing Jira as the team/org planning system — Jira remains the source of truth for sprints, story points, velocity, and cross-team visibility.
- Not building a Kanban board or project management UI — the queue is a flat, ordered list (active → next → inbox).
- Not auto-assigning work or auto-starting tasks — the queue is advisory, not directive.
- Not organizational scope — this is a personal (single-engineer) queue; team-level aggregation is future work.
- Not real-time sync on every Jira state change (Phase 1) — polling or manual refresh is acceptable initially.

### 1.4 Acceptance Criteria

**Phase 1 — Local structured queue:**

- [x] A `queue.md` file exists in the Chronicle repo at a stable path (e.g. `chronicles/queue.md`), with structured sections: Active, Next Up, Inbox.
- [x] Each item supports inline tags: `[jira:TICKET-123]`, `[prd:0003/2]`, `[due:YYYY-MM-DD]`, `[follow-up]`, `[blocked:reason]`.
- [x] A CLI command or skill (`/queue`) displays the current queue, ordered by priority signals (due date, dependency, momentum).
- [x] Items can be added from any source: hand-typed, CLI flag, notes-file convention.
- [x] The queue is git-tracked and survives regeneration (authoritative input, like notes in PRD-0003).

**Phase 2 — Source integrations (pull) & promote (push):**

- [ ] Jira assignments for the user are pulled into the queue inbox (ticket key, title, due date, story points).
- [ ] Unbuilt PRD phases (⬜ in the README table) are auto-populated as queue items with `[prd:NNNN/N]` tags.
- [ ] Session follow-ups (explicit "TODO" or "next step" markers in Claude sessions) are captured into the queue inbox.
- [ ] Duplicate items from different sources are deduplicated by their external ref (Jira key, PRD phase id).
- [ ] `chronicle queue promote` creates an external issue from a queue item and writes the ref back: Rosetta items → GitHub Issues (`gh issue create` in the relevant private repo, tagged `[gh:repo#N]`); enterprise-track items → Jira ticket (tagged `[jira:KEY]`). Target inferred from context (`[prd:*]` → GitHub) or explicit `--target github|jira`.

**Phase 3 — Auto-close loop with Chronicle:**

- [ ] When Chronicle records activity matching a queue item (commit with Jira key in scope, PRD phase shipped, follow-up completed), the item is automatically marked done.
- [ ] Closed items are moved to a "Done" section (or archived) with a link to the Chronicle evidence that closed them.
- [ ] The agent can suggest "what's next?" by reading the queue, considering priority signals, and recommending the top item with context.

## 2. Users & Motivation

Serves the engineer first — "one place to look, one command to ask." Removes:

- **Context re-derivation tax:** no more opening Jira, scanning a notepad, and
  remembering mental state at session start.
- **Lost follow-ups:** "re-run backfill with correct scope" never falls through
  because it's captured structurally, not ephemerally.
- **Fragmented intake:** a Slack request, a Jira assignment, and an idea from a
  1:1 all land in the same queue.
- **Silent staleness:** PRD phases that are Accepted but unbuilt surface
  automatically rather than waiting for someone to re-read the README.

Later serves agents: the queue is the structured input for an autonomous
engineering assistant that can triage, prioritize, and eventually execute work
without the engineer manually curating.

## 3. Approach

The queue is a **structured file** in the Chronicle repo — authoritative input,
like notes (PRD-0003). It is human-editable (plain Markdown) and
machine-readable (tagged items with known semantics).

**The loop:**

```
Sources → Intake → Queue → Work → Chronicle → Close → Archive
   ↑                                              |
   └──────────────────────────────────────────────┘
         (Chronicle completion auto-closes queue items)
```

**Queue structure:**

```markdown
## Active
<!-- Currently in progress. Max 1–3 items — WIP limit for focus. -->
- [ ] Fix quota limit resolution field mismatch [jira:PROJ-72] [due:2026-07-24]

## Next Up
<!-- Prioritized. The agent suggests from here when you ask "what's next?" -->
- [ ] Build artifact linking (PRD-0006 Phase 1) [prd:0006/1]
- [ ] Investigate CDN host-header validation [jira:PROJ-88] [due:2026-07-28]
- [ ] Push personal chronicle to remote [follow-up]

## Inbox
<!-- Unsorted. Triage into Next Up or discard. -->
- [ ] Think about Wayfinder query layer over promoted knowledge [idea]
- [ ] Re-run backfill with correct $CHRONICLE_PROJECT [follow-up]

## Done
<!-- Auto-closed by Chronicle. Archived weekly. -->
- [x] Calendar ingest adapter (PRD-0003 Phase 2) [prd:0003/2] → 2026-07-23
- [x] Clobber guard (PRD-0005 Phase 1) [prd:0005/1] → 2026-07-23
```

**Priority signals (for agent-suggested ordering):**

| Signal | Source | Weight |
|--------|--------|--------|
| Due date | Jira `dueDate`, explicit `[due:...]` | Hard deadline — urgent |
| Blocked-by resolved | Another item completed that this depended on | Unblocked → bubble up |
| Momentum | Chronicle shows recent activity in the same repo/area | Adjacent work = low context-switch |
| WIP limit | Only 1–3 Active items at once | Finish before starting |
| PRD phase order | Phase N before Phase N+1 within a PRD | Structural dependency |
| Staleness | Item in Next Up for >N days with no activity | Escalate or discard |

**Integration with Jira (Phase 2):**

Jira is read-only from the queue's perspective. The queue pulls assignments in
(creates items); it does not push status back. The engineer closes Jira tickets
in Jira (or an agent does on their behalf as a future automation). The queue's
`[jira:KEY]` tag is the correlation key for dedup and auto-close.

## 4. Data Contracts

```ts
/** A single item in the personal work queue. */
export interface QueueItem {
  /** Stable id (content-hash or source ref). */
  id: string;
  /** One-line description of the work. */
  title: string;
  /** Current state in the queue. */
  state: 'active' | 'next' | 'inbox' | 'done';
  /** External references for dedup and auto-close. */
  refs: QueueRef[];
  /** Priority signals attached to this item. */
  signals: QueueSignal[];
  /** When the item was added to the queue. */
  addedAt: string;
  /** When moved to done (if applicable). */
  closedAt?: string;
  /** Chronicle evidence that closed it (activity id). */
  closedBy?: string;
}

export interface QueueRef {
  type: 'jira' | 'prd' | 'pr' | 'follow-up' | 'idea' | 'slack';
  /** External id: Jira key, PRD phase id, PR number, etc. */
  key: string;
  /** Optional URL for linking. */
  url?: string;
}

export interface QueueSignal {
  type: 'due' | 'blocked' | 'momentum' | 'dependency';
  value: string; // ISO date, blocking reason, repo name, etc.
}

/** Read/write the queue file. */
export interface IQueueStore {
  read(repoPath: string): Promise<QueueItem[]>;
  write(repoPath: string, items: QueueItem[]): Promise<void>;
  append(repoPath: string, item: QueueItem): Promise<void>;
}
```

## 5. Constraints & Dependencies

- **Architecture:** Handler / Service / Repository + InversifyJS. Queue file I/O
  is a repository (`QueueStore`); triage/prioritization logic is a service;
  the CLI or skill is the handler.
- **Privacy (ADR-0002):** the queue is private (personal Chronicle repo). Jira
  keys are non-sensitive; Jira content (titles, descriptions) may reference
  internal systems — same privacy posture as session titles.
- **Depends on:** PRD-0002 (structured sidecar — the activity data auto-close
  matches against), PRD-0003 (authoritative-input file pattern — queue.md follows
  the same contract), PRD-0005 (clobber guard — queue must not be overwritten).
- **Relates to:** Jira (read-only pull of assignments), Wayfinder (future: query
  "what should I do next?" against the queue + context).
- **Team standard:** queue.md is human-editable Markdown with inline tags — same
  as notes. No proprietary format.

## 6. Risks & Open Questions

- **Jira sync frequency:** real-time webhooks vs. periodic polling vs. on-demand
  pull? Polling is simplest; webhooks require infrastructure. Start with
  on-demand (`/queue sync`) and add periodic later.
- **Auto-close confidence:** what constitutes "Chronicle recorded matching
  activity"? A commit with `PROJ-72` in scope → close `[jira:PROJ-72]`. A PRD
  phase marked ✅ → close `[prd:NNNN/N]`. But fuzzy matches (session title
  resembles queue item) risk false closes.
- **Queue bloat:** without pruning, inbox grows unbounded. Need a staleness
  policy (items untouched for N days → prompt to discard or re-prioritize).
- **Multi-queue:** should there be one queue per project/workspace, or one global
  personal queue? Start global (one per Chronicle repo); scope later if needed.
- **Agent autonomy level:** Phase 3 has the agent suggesting "what's next?" —
  should it also be able to reorder the queue, or only suggest? Start
  suggest-only; let the engineer reorder manually.
- **Jira two-way sync:** closing a queue item does NOT close the Jira ticket
  (by design). Should there be an opt-in "also transition Jira to Done"? Risky
  (premature close, wrong workflow transition). Defer to a future automation PRD.

## 7. Rollout & Phases

1. ✅ **Phase 1 — Local structured queue:** the `queue.md` file, inline tags,
   hand-population, CLI display (`/queue`), git-tracked and authoritative. The
   engineer manually adds and closes items. Establishes the format and the habit.
   *(shipped)*
2. **Phase 2 — Source integrations (pull) & promote (push):** Jira adapter (read
   assignments into inbox), PRD phase scanner (unbuilt phases → inbox), session
   follow-up extractor (TODO markers → inbox). Dedup by external ref. `queue
   promote` pushes items outward: Rosetta → GitHub Issues (private repos),
   enterprise track → Jira.
3. **Phase 3 — Auto-close loop with Chronicle:** when Chronicle records matching
   activity, mark queue items done automatically. Agent suggests "what's next?"
   by reading the queue and applying priority signals.

## 8. Future Considerations

- **Agent-driven triage:** incoming items auto-sorted into priority order based
  on signals, without human intervention.
- **Jira write-back:** opt-in automation that transitions Jira tickets when the
  queue item closes (requires workflow mapping per project).
- **Sprint planning integration:** at sprint boundaries, the agent suggests which
  inbox items to pull into Next Up based on capacity and due dates.
- **Cross-queue dependencies:** "my item is blocked by someone else's item" —
  requires visibility into other engineers' queues (team-level feature).
- **Wayfinder "what's next?" query:** natural language interface over the queue
  that considers context, energy level, and available time window.
- **Slack intake:** a DM or channel message tagged with a keyword lands directly
  in the queue inbox, no context-switching required.
