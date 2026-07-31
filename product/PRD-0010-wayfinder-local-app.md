---
id: PRD-0010
title: Wayfinder Local App
status: Proposed
date: 2026-07-24
owner: Russ Watson
related_adrs: [ADR-0002]
related_specs: []
supersedes: null
---

# PRD-0010: Wayfinder Local App

> A local-first desktop application that gives non-engineers (PMs, product owners,
> executives) the same knowledge capture, contribution, and query capabilities that
> engineers get through Chronicle + Claude Code — routed through the Claude API,
> backed by invisible git for a full audit ledger.

## 1. Overview & Goals

### 1.1 Purpose

Today, Rosetta's knowledge loop is engineer-only: Chronicle captures work via CLI
and git; promotion, queues, and queries all assume a terminal-native user. But the
most authoritative org knowledge — strategy, decisions, priorities, constraints —
lives in the heads of PMs, product owners, and executives who will never use a CLI
or push to GitHub.

Meanwhile, non-engineers often bounce between chat tools with no durable memory
and no connection to the team's knowledge graph. Engineers get Chronicle + Claude
Code; everyone else gets tabs and copy-paste.

Wayfinder closes that gap:

1. **Knowledge contribution:** non-engineers get their own personal chronicle,
   notes, and queue — same format, same coherence protocol, same promotion path to
   org knowledge. They just don't see git.
2. **AI interface:** every AI interaction flows through the Claude API from the local
   app — credentials stay out of the webview. Wayfinder is "Claude for everyone
   who isn't living in a terminal."
3. **Knowledge consumption:** natural-language queries over org knowledge (PRDs,
   artifacts, promoted chronicles) with evidence-backed answers.

The key architectural insight: **bundled git** (invisible to the user, like
Sourcetree) gives non-engineers the same ledger, audit trail, sync, conflict
resolution, and offline-first capability as engineers — without exposing any of it.
Every note save is a commit. Every sync is a push. Every promotion is a PR. The
user sees "Wayfinder." Under the covers, the persistence layer is identical.

### 1.2 Goals

- Provide non-engineers a first-class knowledge capture and query experience equivalent to what engineers get via Chronicle + Claude Code.
- Route AI interactions through the Claude API from the local app (no Anthropic API keys in the webview).
- Use invisible bundled git as the persistence/ledger layer: full audit trail, offline-first, proven sync, identical format to engineer chronicles.
- Enable non-engineers to contribute to org knowledge through the same coherence protocol (PRD-0009) — their contributions are first-class.
- Local-first: the app and all data live on the user's machine. Sync is opt-in and git-native.
- Ship as a native desktop app (Tauri) — fast, lightweight, no browser dependency.

### 1.3 Non-Goals

- Not a replacement for Claude Code — engineers continue using CLI/editor. Wayfinder is the parallel surface for non-engineers.
- Not a hosted/SaaS product — no server infrastructure, no user database, no web login (Phase 1). Remote access is a future consideration.
- Not a project management tool — does not replace Jira for sprint planning, story points, or velocity tracking. Complements it as a personal/knowledge layer.
- Not real-time collaboration — each user has their own local repo. Sharing happens through promotion to the org repo, not simultaneous editing.
- Not building a custom LLM — routes to Claude via the Claude API. The intelligence is Claude; Wayfinder is the delivery mechanism + knowledge context.

### 1.4 Acceptance Criteria

**Phase 1 — App shell, personal chronicle, Claude AI:**

- [ ] A Tauri desktop application launches on macOS (Windows/Linux follow-on) with Wayfinder branding.
- [ ] Bundled git binary (libgit2 or embedded git) handles all persistence — the user never interacts with git directly.
- [ ] First-run onboarding provisions a local git repository for the user's personal chronicle (same directory structure: `chronicles/`, `chronicles/notes/`, `chronicles/queue.md`).
- [ ] Users can capture notes (free-form Markdown) that are auto-committed to their local chronicle repo.
- [ ] An AI chat interface routes all requests through the Claude API, using `ANTHROPIC_API_KEY`.
- [ ] AI conversations are captured as chronicle activity (same structured format as Claude Code sessions in PRD-0001).
- [ ] Token usage from each reply is surfaced in the UI (input/output counts).

**Phase 2 — Queue, sync, org knowledge query:**

- [ ] Personal work queue (PRD-0007 format) with a GUI: add items, triage, mark done — backed by the same `queue.md` format.
- [ ] Sync to remote: push/pull personal chronicle to a private remote repo (GitHub, CodeCommit). Conflict resolution handled by git merge (surfaced as a simple "resolve" dialog if needed).
- [ ] Query org knowledge: natural-language questions answered by RAG over the org repo (PRDs, promoted artifacts, org chronicles) with cited evidence.
- [ ] Browse org knowledge: tree/search view of promoted artifacts, PRDs, architecture decisions.

**Phase 3 — Contribution, promotion, team visibility:**

- [ ] Promote personal knowledge to org repo via the coherence protocol (PRD-0009) — Wayfinder opens a PR on behalf of the user, invisible git mechanics.
- [ ] Team activity feed: rolled-up view of recent org chronicle entries across contributors (engineers and non-engineers).
- [ ] Cross-role context: a PM can see what engineering activity happened related to their PRD (because Chronicle captured it) without reading commits.
- [ ] Admin/onboarding: provision new users (create their personal repo, configure Claude creds, clone org repo) via a setup flow in the app.

## 2. Users & Motivation

### Primary: Product Managers, Product Owners, Engineering Managers

**Pain today:**

- "What's the status of X?" requires pinging an engineer or navigating issue trackers.
- Meeting notes, decisions, and context live in scattered docs — disconnected from the engineering knowledge graph.
- AI chats don't persist into the team's durable memory.
- No structured personal "second brain" — just tabs and recollection.

**With Wayfinder:**

- Ask "what's the status of the quota limit work?" and get an evidence-backed answer in seconds.
- Capture notes and decisions that feed directly into org knowledge — same coherence, same attribution.
- AI interactions are captured into the chronicle by default.
- Personal queue + notes give them the same "what's next?" clarity that engineers get.

### Secondary: Executives, Directors

**Pain today:**

- Strategic context (why we chose X over Y, what constraint drove a decision) is verbal-only or buried in email. It decays within weeks.
- They have the most authoritative knowledge about org direction but no structured way to contribute it.

**With Wayfinder:**

- A 2-minute note after a decision meeting becomes durable org knowledge that engineers and agents can reference months later.
- Their contributions are _more_ authoritative than dev commits — and the system treats them as such.

### Tertiary: Future Agents

- Agents querying org knowledge get a richer, more authoritative corpus when non-engineers contribute.
- The coherence protocol ensures contributions from all roles are consistent and current.

## 3. Approach

### 3.1 Architecture

```
┌───────────────────────────────────────────────────────┐
│                  Wayfinder App (Tauri)                  │
├────────────────┬─────────────────┬────────────────────┤
│  Note Capture  │  AI Chat        │  Org Knowledge     │
│  Queue Mgmt    │  (Claude)      │  Query + Browse    │
│  Chronicle     │                 │  Promote           │
└───────┬────────┴────────┬────────┴────────┬───────────┘
        │                 │                 │
        ▼                 ▼                 ▼
┌──────────────┐ ┌─────────────────┐ ┌─────────────────┐
│ Bundled Git  │ │ the Claude API     │ │ Org Repo (git)  │
│ (libgit2)    │ │ (Claude model)  │ │ (clone + pull)  │
│              │ │                 │ │                 │
│ Local repo:  │ │ - Chat          │ │ - PRDs          │
│ - chronicles │ │ - RAG queries   │ │ - Artifacts     │
│ - notes      │ │ - Summarization │ │ - Org chronicle │
│ - queue      │ │                 │ │                 │
└──────────────┘ └─────────────────┘ └─────────────────┘
```

### 3.2 Bundled Git as Invisible Ledger

The user sees:

- "Save note" → happens instantly
- "Sync" button → their data backs up
- "Promote" → knowledge appears in org
- "History" → they can see their past activity

Under the covers:

- "Save note" → `git add` + `git commit` (auto-message with timestamp)
- "Sync" → `git push` to their private remote
- "Promote" → `git push` to org repo + open PR (coherence gate)
- "History" → `git log` formatted into a timeline

Conflicts (rare — single user per repo) surface as: "Your remote has changes.
Review?" with a simple accept/merge dialog. No git terminology exposed.

Implementation: **libgit2** (Rust bindings via `git2` crate) compiled into the
Tauri binary. No external git installation required. Same approach as Sourcetree,
GitHub Desktop, VS Code's built-in git.

### 3.3 Claude Integration

```
Wayfinder Chat → Claude API (Claude) → Response
     ↓                                      ↓
  Capture input/output              Display to user
  as chronicle activity
```

Credentials: `ANTHROPIC_API_KEY` in the process environment. The Rust transport
reads the key and never exposes it to the webview.

Usage display: each reply returns model id and token counts so the UI can show
what a request cost.

### 3.4 Non-Engineer Chronicle Format

Identical to engineer format:

```
chronicles/
├── 2026-07-24.md          (rendered daily summary — generated)
├── notes/
│   └── 2026-07-24.md      (user's notes — authoritative input)
├── queue.md                (personal work queue)
└── .data/
    └── 2026-07-24.json    (structured sidecar — machine-readable)
```

Activity sources differ (no git commits, no Claude Code sessions), but the
structure is the same:

| Engineer sources     | Non-engineer sources                    |
| -------------------- | --------------------------------------- |
| Git commits          | —                                       |
| Claude Code sessions | Wayfinder AI conversations              |
| Jira (shared)        | Jira (shared)                           |
| Calendar (shared)    | Calendar (shared)                       |
| Manual notes         | Manual notes                            |
| —                    | Meeting recordings/transcripts (future) |

### 3.5 First-Run Onboarding

1. User opens Wayfinder for the first time.
2. App creates a local directory (e.g. `~/Wayfinder/chronicle/`) and initializes a git repo.
3. User confirms `ANTHROPIC_API_KEY` is available in the environment.
4. (Optional) User connects a remote for sync — private GitHub repo, self-service.
5. (Optional) User clones the org repo for knowledge queries.
6. Ready. First note save creates the first commit.

### 3.6 Tech Stack

| Layer         | Technology                      | Why                                                                           |
| ------------- | ------------------------------- | ----------------------------------------------------------------------------- |
| App shell     | Tauri 2.x (Rust + web frontend) | Native performance, small binary, no Electron overhead, access to system APIs |
| Frontend      | React + TypeScript              | Team familiarity, component ecosystem                                         |
| Git           | libgit2 via `git2` Rust crate   | No external dependency, compiled in, full git capability                      |
| AI            | Anthropic Messages API (HTTPS)  | ANTHROPIC_API_KEY stays in the native process                                 |
| Markdown      | Remark/unified ecosystem        | Parse and render chronicle format                                             |
| Local storage | Filesystem (git-managed)        | No database, no migration headaches, portable                                 |

## 4. Data Contracts

```ts
/** Wayfinder user profile — stored in app config, not in the chronicle. */
export interface WayfinderProfile {
  displayName: string;
  email: string;
  role: "engineer" | "pm" | "executive" | "other";
  claudeModelId: string;
  /** Local path to personal chronicle repo. */
  chroniclePath: string;
  /** Remote URL for sync (optional). */
  remoteUrl?: string;
  /** Local path to org repo clone (optional). */
  orgRepoPath?: string;
}

/** A Wayfinder AI conversation turn — maps to chronicle Activity. */
export interface WayfinderConversation {
  id: string;
  startedAt: string;
  endedAt?: string;
  turns: WayfinderTurn[];
  /** Derived title for the conversation (same as Claude Code session title). */
  title: string;
  /** Token usage for cost tracking. */
  usage: { inputTokens: number; outputTokens: number };
}

export interface WayfinderTurn {
  role: "user" | "assistant";
  content: string;
  timestamp: string;
}

/** Auto-commit metadata — the invisible git commit message. */
export interface LedgerEntry {
  action: "note-save" | "queue-edit" | "conversation-end" | "sync" | "promote";
  timestamp: string;
  summary: string;
}
```

## 5. Constraints & Dependencies

- **Architecture:** Tauri (Rust backend + web frontend). The Rust side handles git operations, Claude API calls, and filesystem access. The web frontend handles UI only.
- **AI transport:** all AI traffic routes through the Claude API. No Anthropic API keys in the webview.
- **Privacy (ADR-0002):** personal chronicle is private (user's machine + their private remote). Org knowledge is shared. Same boundary as engineer chronicles.
- **Depends on:** PRD-0006 (artifact promotion — Wayfinder users promote too), PRD-0007 (queue format — reused as-is), PRD-0009 (coherence protocol — gates non-engineer promotions identically).
- **Git provisioning:** personal repos for non-engineers must be provisionable without the user knowing git. Self-service flow preferred.
- **Claude credentials:** users need `ANTHROPIC_API_KEY` with Messages API access.
- **Org repo access:** read access to the org repo for knowledge queries. Write access (fork + PR) for promotion. Standard git permissions.

## 6. Risks & Open Questions

- **Tauri maturity:** Tauri 2.x is production-ready but less ecosystem support than Electron. Tradeoff: much smaller binary and better performance vs. fewer off-the-shelf plugins.
- **libgit2 limitations:** handles 99% of git operations but edge cases (interactive rebase, complex merges) may surface. Mitigated by: single-user repos rarely conflict; auto-commit messages are formulaic; merge strategy is simple (theirs/ours with user prompt).
- **Claude model access:** requires a valid ANTHROPIC_API_KEY with Messages API access.
- **Onboarding UX:** non-engineers won't tolerate a complex setup. First-run must be <3 minutes: install, open, point at chronicle, done. Git repo creation is invisible.
- **Adoption:** PMs and executives adopt tools that solve an immediate pain. The "ask me about project status" query capability is the hook. Note-taking and chronicle are the habit that follows. Sequence the rollout accordingly.
- **Org repo size at scale:** as more people promote, the org repo grows. Git handles this well (shallow clones, sparse checkout for Wayfinder users who only need recent data). Not a Phase 1 concern.
- **Meeting transcript ingestion:** a future high-value source for non-engineers. Requires audio processing pipeline (Whisper/Claude transcription). Explicitly deferred — notes are the manual equivalent for now.

## 7. Rollout & Phases

1. **Phase 1 — App shell, personal chronicle, Claude AI:** Tauri app with
   bundled libgit2, local chronicle repo, note capture (auto-committed), AI chat
   via the Claude API, session capture as chronicle activity. Wayfinder branding. macOS
   first.
2. **Phase 2 — Queue, sync, org knowledge query:** personal queue GUI, remote
   sync (push/pull with simple conflict UX), RAG-based org knowledge queries with
   evidence citation, org repo browse/search.
3. **Phase 3 — Contribution, promotion, team visibility:** promote to org via
   coherence protocol, team activity feed, cross-role context ("what engineering
   work happened on my PRD?"), user provisioning/onboarding admin flow.

## 8. Future Considerations

- **Windows/Linux:** Tauri supports all three platforms. macOS first (team's primary OS), cross-platform in a follow-on release.
- **Meeting transcript ingestion:** auto-capture from Zoom/Teams recordings via transcription → chronicle notes. High value for PMs who live in meetings.
- **Mobile companion:** read-only view of queue + org knowledge on iOS/Android. Tauri doesn't target mobile natively — would be a separate React Native app reading the same git-backed data.
- **Agent-as-user:** an autonomous agent running inside Wayfinder that triages the user's inbox, suggests "what's next," drafts notes from calendar events, and promotes knowledge on the user's behalf (with approval).
- **Federated org repos:** multiple teams with separate org repos, cross-referenced by the coherence protocol. Wayfinder queries span all accessible repos.
- **Link sharing:** share a specific chronicle entry or artifact with a colleague via link, with auth deferred to a later design.
- **Spend visibility:** optional rollup of Claude token usage across Wayfinder sessions for personal budgeting.
