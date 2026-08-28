---
id: chronicle-v1-readiness-2026-08-28
title: Chronicle V1 readiness — first allowlisted export
date: 2026-08-28
status: Working record
authorship: collaborative
---

# Chronicle V1 readiness — first allowlisted export

Sanitized return from the Build Charter’s V1 readiness review.

Private operator state stays outside git
(`~/.local/share/rosetta/chronicle-build/CURRENT-STATE.md`).
This note has **no** export paths, archive hashes, conversation or
node ids, attachment ids, message text, or titles.

It does **not** widen PRD-0027, unfreeze v0.1 `Activity`, authorize
interpretation, or authorize ChatGPT Desktop ingest.

---

## What exists (operator can do this now)

V1 raw observe is on `rosetta_chronicle` `main` (#30 file scopes, #31
directory scopes):

```text
observe-init --data-dir --scope --path <file-or-dir>
→ observe / watch [--once]
→ hash → copy-if-new → receipt
→ vault-status / vault-resolve
→ observe-stop / observe-resume
→ forget-scope
```

Directory walk is recursive under the allowlisted root only; symlinks
that escape it are skipped. `observe` / `watch` stdout is counts, not
source paths.

A **RED-authorized** ChatGPT **data-export** directory was observed
into an operator-private vault (OS disk encryption; mode `0700`).
Re-observe of the unchanged tree was all duplicate. STOP and
`forget-scope` were not exercised on that copy.

The same export was then **catalogued** with `import-chatgpt` into a
caller-chosen private directory. The graph is source structure
(topology, clocks, roles/types, attachment presence). It is not an
archive backup and not `Activity`. Re-import of the same archive hash
was `already-present`.

| Catalog (sanitized) | Count |
| ------------------- | ----- |
| Conversations       | 367   |
| Nodes               | 7122  |
| Nodes with a message | 6755 |
| Attachment refs     | 383   |
| Attachment blobs missing from the export | 18 |
| Unsupported records | 18 |
| Conversation shards | 4 |

Roles present on message nodes: `user`, `assistant`. Content types
present: `text`, `multimodal_text`, `reasoning_recap`, `thoughts`.
Every conversation in the graph has a `currentNodeId`. Message text,
titles, and display filenames are not in the graph (verified by key
scan on the private file).

## What the operator cannot do yet

- Treat the vault as a backup. One laptop. Desktop export is a second
  copy the operator already had. Neither is a backup strategy.
- Forget at byte-level, crypto-shred, or claim vendor deletion.
  V1 forget is **scope-only** of copies Chronicle wrote.
- Observe ChatGPT **Desktop** (or Cursor/Claude) until a specific path
  is allowlisted.
- Interpret this corpus (`interpret-source`) without a new RED
  envelope (provider, what leaves the machine, what is stored).
- Rebuild conversation bodies from the graph if the export/vault is
  gone.

## Limits the pilot made concrete

1. Receipts are append-only per file per pass (including duplicates).
2. Same bytes under two names in one tree store once (copy-if-new).
3. Silent reacquisition from a new locator remains a documented
   limitation.
4. `vault-status` still prints scope paths (operator-local only).
5. `watch` on an unchanged export zip is noise.

## Decisions this review does not reopen

Path over destination. Source ≠ interpretation ≠ evaluation ≠ current
understanding. No biography. No auto-promotion. v0.1 `Activity` /
`getActivity` frozen. No Capture Engine. No home-directory sweep. No
Chronicle-owned app encryption for V1. No second allowlist required
to call this increment done.

## Next specimen (authorized to locate, not to ingest)

After a correct catalog of a ChatGPT **data-export** dump: find where
**ChatGPT Desktop** stores prompts, responses, and tool uses on this
host.

Public prior art (not a capture): macOS app data commonly lives under
`~/Library/Application Support/com.openai.chat/`. Early builds stored
plaintext conversation JSON; later builds encrypt `conversations-v2`
/ `conversations-v3` trees with an app-held key. A content-blind
existence check on this host: that Application Support root is
present; `conversations-v3*` directories exist. **No conversation
files were opened. No Desktop path is allowlisted. No bytes were
copied into the vault.**

Next work on that specimen:

1. Content-blind inventory (directory kinds, file types, encryption
   vs plaintext) — names that are account or conversation ids stay
   private.
2. If anything useful is plaintext, propose an **explicit allowlist**
   and wait for RED.
3. If the live store is ciphertext only, the data-export remains the
   honest readable source; Desktop may still be a change-detection
   signal later, not a second body store.

Do not decrypt. Do not home-sweep. Do not enable Cursor/Claude
capture as a substitute.
