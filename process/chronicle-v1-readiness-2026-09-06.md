---
id: chronicle-v1-readiness-2026-09-06
title: Chronicle V1 readiness — turn-on closeout
date: 2026-09-06
status: Working record
authorship: collaborative
---

# Chronicle V1 readiness — turn-on closeout

Hi Addi.

Sanitized return from the Build Charter’s V1 readiness review, after
`chronicle start` landed and a second allowlisted source (Cursor JSONL)
was observed into the same private vault.

Private operator state stays outside git
(`~/.local/share/rosetta/chronicle-build/CURRENT-STATE.md`).
This note has **no** export paths, archive hashes, conversation or
node ids, attachment ids, message text, titles, or project slugs.

It does **not** widen PRD-0027, unfreeze v0.1 `Activity`, authorize
interpretation, ChatGPT Desktop ingest, a plaintext search index, or
putting the vault in a git ledger.

The 2026-08-28 note remains the first-catalog checkpoint:
[`chronicle-v1-readiness-2026-08-28.md`](chronicle-v1-readiness-2026-08-28.md).

---

## Charter done-when (this increment)

| Claim | Status |
| ----- | ------ |
| Configure an explicit scope | Yes — `observe-init` |
| Automatically observe changes | Yes — `start` / `watch` (`--once` is one pass) |
| Preserve changed bytes | Yes — hash, copy-if-new |
| Deduplicate unchanged | Yes — second pass is `duplicate` except growing files |
| Honest provenance / clocks | Yes — observe-time receipts; not authored event time |
| Resolve retained evidence | Yes — `vault-resolve` by content hash |
| STOP | Yes — `observe-stop` / `observe-resume` |
| forget-scope | Command exists. **Not exercised on this live vault.** |
| Survive restart | Data is on disk. The poller is restarted by the operator (or a local supervisor). Not launchd. |
| Canonical vs rebuildable | Vault objects are canonical. Graphs and conversation view are rebuildable. Views write nothing. |
| No Activity | Yes — observe does not emit Daily Chronicle `Activity` |

## What the operator can do now

```text
observe-init --scope <id> --path <file-or-dir>
→ chronicle start [--once]
→ vault-status / vault-resolve
→ observe-stop / observe-resume
→ forget-scope
→ chatgpt-conversation-view / chatgpt-conversation-locate
```

`start` is the V1 turn-on alias for `watch`. Default data-dir is
`~/.local/share/rosetta/chronicle/default` unless `--data-dir` or
`$CHRONICLE_DATA_DIR` is set.

Two source kinds are allowlisted on this host (operator-authorized):

- ChatGPT **data-export** directories (prior RED pilot)
- Cursor **agent-transcript** directories (Rosetta workspace + one
  named project family; not `~/.cursor` as a whole)

A ChatGPT source-graph catalog still exists beside the vault
(topology and clocks only; no titles or parts). Conversation view
and locate read those snapshots only. They do not read vault bytes.

## What this is not

- A backup. One laptop. OS disk encryption. Vault objects cannot be
  rebuilt if the disk is gone. Documented V1 limit — no cloud backend.
- A git ledger. v0.1 Daily Chronicle wrote synthesized markdown to
  `CHRONICLE_REPO`. That remains the right persistence for a chosen,
  reviewable diary — not for raw vault bytes. Clone is publication.
- A search index. One-shot scan of vault objects works. FTS is still
  an operator decision (purpose, forget, host indexers) — not a
  performance gate on this corpus.
- ChatGPT Desktop ingest. Locate (2026-08-28) found ciphertext
  `conversations-v3` `*.data`. Do not decrypt.
- A Cursor source graph, `interpret-source` on this corpus, npm /
  Homebrew, or app encryption.

## Limits this increment made concrete

1. **`start` without `--once` polls every 2 seconds.** Receipts are
   append-only per file per pass, including duplicates. Leaving the
   stock poller up will grow the receipt directory without new
   objects. Capture left running on this host is **periodic
   `start --once`**, not the 2s loop.
2. Growing files (append-only JSONL) store a new object when the
   hash changes. That is correct successive observation.
3. Same bytes under two names store once (copy-if-new).
4. Silent reacquisition from a new locator remains a documented
   limitation. forget-scope is scope-only of copies Chronicle wrote.
5. `vault-status` still prints scope paths (operator-local only).
6. Graph import is still a separate catalog step. `start` vaults
   bytes; it does not run `import-chatgpt`.

## Decisions this review does not reopen

Path over destination. Source ≠ interpretation ≠ evaluation ≠
current understanding. No biography. No auto-promotion. v0.1
`Activity` / `getActivity` frozen. No Capture Engine. No
home-directory sweep. No Chronicle-owned app encryption for V1.
Git for selected ledger artifacts later; not for the vault.

## Next (not authorized unless the operator asks)

- More Cursor slugs
- A Cursor source-graph / conversation view
- Plaintext FTS of vault bytes
- `interpret-source` on this corpus
- Desktop ingest or decrypt
- Publishing or remoting the vault
- Changing the stock `start` interval / receipt policy in the engine
