---
id: PRD-0012
title: Wayfinder AI Workspace
status: Proposed
date: 2026-07-26
owner: Russ Watson
related_adrs: [ADR-0003, ADR-0004]
related_specs: []
supersedes: null
---

# PRD-0012: Wayfinder AI Workspace

> A persistent, context-aware AI pair inside Wayfinder that has access to the
> user's chronicle, can take actions on their behalf, and remembers the thread
> of work across sessions — giving non-engineers the same AI-augmented workflow
> that engineers have today through Chronicle + Claude Code.

## 1. Overview & Goals

### 1.1 Purpose

Today, engineers using Chronicle + Claude Code have a deeply capable AI pair:
it reads their files, writes notes, updates queues, runs code, and maintains
context across an entire day's work. The AI knows what the engineer is doing
because it is operating inside the same workspace.

Non-engineers using Wayfinder today have a chat panel — stateless, single-turn,
no file access, no memory between sessions. Every conversation starts from zero.
Asking "what did I work on this week?" returns "I don't have access to your
activity." The AI is present but blind.

The gap is not a UI problem. It is a capability gap: the current Wayfinder chat
has no access to the user's chronicle, no ability to take actions, and no
persistent memory. Closing that gap is what this PRD defines.

The Wayfinder AI Workspace gives non-engineers — PMs, product owners, executives,
community coordinators — an AI pair that:

- Knows their chronicle (notes, daily activity, queue)
- Can take actions on their behalf (capture a note, update a queue item, promote
  an artifact)
- Remembers prior conversations as part of the chronicle
- Answers questions grounded in real data, not hallucination
- Routes every AI call through the Claude API

This is "Chronicle + Claude Code for everyone who isn't an engineer." The
experience is conversational; the stack underneath is the same.

### 1.2 Goals

- Give non-engineers a persistent AI pair that operates inside their chronicle
  context, not alongside it.
- Enable action-taking: the AI can write, update, and promote — not just answer.
- Store conversation history as a first-class chronicle artifact so sessions are
  durable, searchable, and part of the knowledge ledger.
- Route all AI traffic through the Claude API from the local app.
- Surface prior conversation context in new sessions so the AI does not start
  from zero each time.
- Make the workspace feel like a natural extension of the Wayfinder app, not a
  separate product.

### 1.3 Non-Goals

- Does not replicate the full Claude Code terminal experience (file editing,
  running tests, shell commands) — the Wayfinder AI Workspace is knowledge-domain
  only.
- Does not expose git directly to the user — all chronicle mutations go through
  the existing invisible-ledger pattern (ADR-0003).
- Does not build a general-purpose agent framework — tool surface is scoped to
  Wayfinder's domain (chronicle, queue, notes, org knowledge).
- Does not replace the existing free-chat and org-query modes — those remain as
  lighter-weight surfaces; the workspace is opt-in for deeper sessions.
- Does not require internet or a cloud account beyond the existing Claude API access
  credential (local-first, ADR-0003).

### 1.4 Acceptance Criteria

- [ ] The workspace maintains a conversation thread that persists across app
      restarts, stored in `chronicles/workspace/<date>.md`.
- [ ] On session start, the AI is given a system prompt that includes: today's
      chronicle, recent notes, queue Active/Next Up items, and the last N turns
      of prior conversation.
- [ ] The AI can invoke at least three chronicle actions: capture note, toggle
      queue item, answer from org knowledge.
- [ ] Each AI action that mutates the chronicle produces a git commit
      (`workspace-action: <description>`).
- [ ] Conversation history is rendered in the UI as a scrollable thread with
      clear role labels and timestamps.
- [ ] The user can start a new session (clears in-memory thread; prior sessions
      remain in the chronicle).
- [ ] All AI traffic routes through the Claude API; the Claude footer is present.
- [ ] Unit tests cover: context assembly, action dispatch, history serialisation.

## 2. Users & Motivation

**Primary: non-engineer Rosetta users** — PMs, product owners, executives,
community coordinators. Today they interact with AI in disposable chat tabs with
no durable memory. They don't have a persistent workspace. They lose context
between sessions. They can't ask "what did I work on this week?" and get a real
answer.

**Secondary: engineers** who want a GUI complement to Chronicle + Claude Code —
e.g. a PM-mode session for strategic thinking, separate from the terminal flow.

**The pain being removed:** starting every AI conversation from zero. The
cognitive cost of re-establishing context ("I'm working on X, here's the
background...") is high and compounds across a week. The workspace eliminates
that cost — the AI already knows.

**The pitch moment:** a PM opens Wayfinder before a stakeholder meeting and asks
"summarise what we shipped this week and what's at risk." The AI draws from the
week's chronicle entries, the queue, and the org knowledge base — and answers in
10 seconds with citations. No copy-paste. No context re-establishment. That's the
moment the product story is built around.

## 3. Approach

### 3.1 Context assembly

On each workspace turn, the AI receives a system prompt assembled from:

```
1. Role definition: "You are Wayfinder, a knowledge assistant..."
2. Today's chronicle (chronicles/<today>.md) — the activity log
3. Today's notes (chronicles/notes/<today>.md) — manual entries
4. Queue Active + Next Up items (chronicles/queue.md)
5. Recent org knowledge summary (optional, toggled by user)
6. Last N conversation turns from prior sessions (chronicles/workspace/*.md)
```

Context is assembled by a `WorkspaceContextService` that reads from the
chronicle repo. Stale context (older sessions) is summarised or truncated to
stay within Claude's context window.

### 3.2 Action tools

The AI can invoke a fixed set of chronicle actions, declared in the system prompt
as tool descriptions. Each tool maps to an existing HSR service call:

| Tool                | Maps to                            | Commit message                         |
| ------------------- | ---------------------------------- | -------------------------------------- |
| `capture_note`      | `NotesService.appendEntry()`       | `workspace-action: captured note`      |
| `toggle_queue_item` | `QueueService.toggleItem()`        | `workspace-action: toggled queue item` |
| `ask_org_knowledge` | `KnowledgeService.ask()`           | _(read-only, no commit)_               |
| `read_chronicle`    | `ChronicleFsRepository.readFile()` | _(read-only)_                          |

Tool invocations are parsed from the model's response (structured JSON block),
executed by the `WorkspaceService`, and the result is fed back as the next
context turn.

Tools are intentionally conservative: the AI cannot delete, overwrite, or push.
All mutations are append-only, matching the existing chronicle contract.

### 3.3 History persistence

Each workspace session is stored as `chronicles/workspace/<date>-<session>.md`:

```markdown
# Workspace Session — 2026-07-26 09:14

## Turn 1

**You:** Summarise what I shipped this week.
**Wayfinder:** Based on your chronicle from 2026-07-21 to 2026-07-26...

## Turn 2

...
```

Sessions are committed to the chronicle on every turn (`workspace-action: turn`),
so they are part of the audit ledger and searchable by the org-knowledge RAG.

### 3.4 UI layout

The workspace is a new panel in the Wayfinder app, distinct from the current
chat panel:

```
┌─────────────────────────────────────────┐
│ AI Workspace              [New session] │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ [scrollable conversation thread]    │ │
│ │                                     │ │
│ │ You: what did I ship this week?     │ │
│ │                                     │ │
│ │ Wayfinder: Based on your chronicle  │ │
│ │ from 2026-07-21 to 2026-07-26, you  │ │
│ │ shipped: ...                        │ │
│ │ [action taken: read chronicle ✓]    │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ [input textarea]          [Send] [Auto] │
│                                         │
│ ROUTED THROUGH CLAUDE API · USAGE ...  │
└─────────────────────────────────────────┘
```

## 4. Data Contracts

```typescript
/** A single workspace conversation turn. */
interface WorkspaceTurn {
  role: "user" | "assistant";
  content: string;
  /** ISO-8601 timestamp. */
  timestamp: string;
  /** Actions the AI invoked this turn, if any. */
  actions?: WorkspaceAction[];
}

/** An action the AI took against the chronicle. */
interface WorkspaceAction {
  tool:
    | "capture_note"
    | "toggle_queue_item"
    | "ask_org_knowledge"
    | "read_chronicle";
  input: Record<string, unknown>;
  /** Result surfaced back to the model. */
  result: string;
  /** Whether the action produced a git commit. */
  committed: boolean;
}

/** A workspace session (one conversation thread). */
interface WorkspaceSession {
  date: string;
  sessionId: string;
  turns: WorkspaceTurn[];
}

/** The assembled context fed to the model on each turn. */
interface WorkspaceContext {
  todayChronicle: string;
  todayNotes: string;
  queueSummary: string;
  recentTurns: WorkspaceTurn[];
}
```

## 5. Constraints & Dependencies

- All AI calls route through the Claude API (ADR-0003). `ANTHROPIC_API_KEY` stays in the native process — never in the webview.
- All chronicle mutations use the existing HSR service layer — no new Rust
  commands needed for Phase 1.
- Context window management is required: a day's chronicle can be 50–200 lines;
  prior session history compounds. Phase 1 uses simple truncation (most-recent N
  turns); Phase 2 introduces summarisation.
- Tool invocation uses structured output parsing from the model response — no
  native Claude tool-use API required in Phase 1 (avoids model-specific
  compatibility concerns).
- Depends on PRD-0010 (Wayfinder app shell) — workspace is a panel inside the
  existing app.
- Depends on ADR-0004 (shared core boundary) — `WorkspaceContextService` shares
  the same chronicle-reading logic as `StandupService` and `KnowledgeService`.
- Conversation history stored in `chronicles/workspace/` — new subdirectory,
  same chronicle repo.

## 6. Risks & Open Questions

- **Context window cost:** assembling today's full chronicle + prior session
  history on every turn could be expensive at Opus pricing. Mitigation: default
  to Sonnet for workspace; let user escalate to Opus for complex sessions.
- **Tool parsing reliability:** Phase 1 uses prompt-based tool invocation (parse
  JSON from model output). If the model doesn't format correctly, the action
  silently fails. Phase 2 should migrate to native Claude tool-use once we
  confirm model compatibility.
- **Session boundary:** when does a session end and a new one begin? Current
  proposal: manual ("New session" button) + automatic rollover at midnight.
  Open question: should sessions be linked (prior session summary injected into
  new session context)?
- **Write safety:** the AI can call `capture_note` and `toggle_queue_item`.
  What prevents it from hallucinating a note capture the user didn't intend?
  Mitigation: Phase 1 requires user confirmation before any mutating action
  executes (a "Wayfinder wants to capture a note — approve?" inline prompt).
- **Privacy:** workspace sessions are stored in the chronicle and committed to
  git. If the chronicle is ever synced to a remote, session transcripts go with
  it. This is by design (full audit trail) but should be surfaced in onboarding.

## 7. Rollout & Phases

1. **Phase 1 — Context-aware conversation:** Workspace panel with persistent
   session history, chronicle context injection (today's activity + notes +
   queue), and conversation storage in `chronicles/workspace/`. No action tools
   yet. Proves the context-assembly and persistence model.

2. **Phase 2 — Action tools:** Add `capture_note`, `toggle_queue_item`, and
   `read_chronicle` tool invocations with user confirmation gate. Every action
   produces a chronicle commit. Proves the AI-as-actor model.

3. **Phase 3 — Session continuity and summarisation:** Inject prior session
   summaries into new session context. Implement context-window management
   (summarise old turns rather than truncating). Add `ask_org_knowledge` tool
   so the workspace can draw from the full org corpus mid-conversation.

## 8. Future Considerations

- **Streaming responses:** the current Claude invocation is request-response.
  Streaming would make long context-assembly turns feel faster and more
  conversational.
- **Voice input:** non-engineers in meetings may prefer dictating rather than
  typing. A push-to-talk mode that transcribes to the workspace input is a
  natural mobile/tablet follow-on.
- **Proactive surface:** the workspace could open with a morning briefing
  ("Here's what's on your plate today") rather than waiting for the user to ask.
  Driven by the standup context-assembly logic already built.
- **Multi-user sessions:** a shared workspace session between a PM and an
  engineer, both contributing turns, both with their own chronicle context.
  The persistence model already supports this (one session file, two
  `repoPath` sources).
- **Native Claude tool-use:** migrate from prompt-parsed tool invocations to
  the Claude Messages API tool-use tool-use spec once model compatibility is confirmed
  across Sonnet/Opus/Haiku.
