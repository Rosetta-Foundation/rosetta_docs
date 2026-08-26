---
id: PRD-0027-raw-source-preservation
title: Raw source preservation — program design (A/B/C)
date: 2026-08-25
prd: PRD-0027
authorship: collaborative
status: Design review
---

# Raw source preservation — design review

Companion to the Build Charter program. Not a Capture Engine spec.
Does not authorize implementation of Gate 4 or encryption. Does not
widen PRD-0027. Floor remains
[privacy-and-forgetting.md](privacy-and-forgetting.md).

Vault-path specimens (VAULT-V0, RESOLVE-COPY-V0, VAULT-CHATGPT-V0,
HIDE-COPY-V0) established only the claims they tested. Do not inflate
them.

---

## Program A — Gate 4 semantics

These are different operations:

```text
stop observing
withhold
delete our stored copy
refuse reacquisition
forget historical versions
forget future versions
```

Chronicle must never claim deletion of vendor systems, Git history it
does not rewrite, third-party backups, clones, caches, or external
archives.

### Recommended V0 (smallest honest)

| Operator intent | V0 meaning | Durable state |
| --------------- | ---------- | ------------- |
| **STOP** | Do not start new observes (global, per kind, or per scope) | Allowlist / acquisition-off config. No content hash. |
| **WITHHOLD** | Encountered or known; do not copy bytes into the vault | Source-native locator or scope on a deny list. Inventory may still run. |
| **FORGET captured content** | Unlink the vault object Chronicle controls | Filesystem delete of `sha256:…` we stored. Receipts may record that the object is gone. |
| **FORGET source-native identity (historical)** | Delete every vault object we already stored for that id/scope | Walk receipts for that source-native id; delete those objects. |
| **FORGET future versions** | WITHHOLD that source-native id / scope going forward | Same deny list as WITHHOLD. |
| **Refuse reacquisition** | V0: refuse **that scope**, not those **bytes** | Deny list of locators / source-native ids. |

**Not in V0:** byte-level “never store these bytes again wherever they
appear.” That needs a content recognizer (a fingerprint). A
git-tracked `SHA256(forbidden bytes)` is **rejected**: it is a
permanent public-or-ledger fingerprint of material the operator asked
to forget.

If byte-level refuse is later required, the recognizer must stay in
the **private** vault (not Chronicle git), and the claim must stay
local: “this operator’s Chronicle will not copy these bytes again,”
not “this content has been erased from the world.”

Silent reacquisition of the **same bytes from a new locator** is a
documented V0 limitation, not a bug to paper over with a git
tombstone.

### What changes if Ready Room picks otherwise

- **Private content tombstone:** prevents byte-level reacquire; keeps
  a local fingerprint; still must not be git-tracked in a public or
  cloneable ledger.
- **Crypto-shred:** see Program B. Destroys readability of ciphertext
  Chronicle encrypted. Does not stop a later observe of the live
  source from creating a new object unless WITHHOLD is also on. Does
  not erase backups Chronicle does not control. Do not claim it does.

### Escalation (YELLOW)

Implementing Gate 4 waits on Ready Room accepting **scope-only V0**
(recommended) vs requiring byte-level refuse or crypto-shred in V0.

---

## Program B — object custody vs key custody

“Vault” is a role. Current practice is a private local
content-addressed directory. That is object custody, not a product
name.

| Option | Role | V0 verdict |
| ------ | ---- | ---------- |
| 1. Local directory + permissions + OS disk encryption (FileVault / LUKS) | Object custody; OS holds the disk key | **Recommended V0.** Already how the specimen vault lives. Chronicle does not encrypt objects. Honest claim: “at rest if the host is.” |
| 2. Application-level encrypted objects | Chronicle encrypts before write | Design-only. New crypto surface. Not GREEN this wave. |
| 3. 1Password (or equivalent) as **key** custody | Keys only | Design-only. Unattended capture must obtain the key somehow; that is a trust decision. |
| 4. restic / borg / equivalent | Encrypted backup of the object store | Design-only. Prefer these over Chronicle-owned backup crypto if backup is required. |
| 5. S3 / B2 / R2 / MinIO | Remote object custody of (preferably encrypted) blobs | Design-only. Same logical `sha256` contract; different physical backend. |
| 6. Secrets manager as the **object** store | Bulk corpus inside 1Password et al. | **Rejected / constrained.** Wrong size, cost, and access model for transcripts, exports, and attachments. |

### Crypto-shredding

Separating key custody from object custody can make ciphertext in a
backup unreadable after key destruction. That is not deletion of
vendor copies, clones, or an unencrypted live source. Do not design
custom cryptography. Do not implement encryption in this wave.

Granularity (one global key vs per-scope keys), rotation, recovery,
machine migration, unattended capture, and catastrophic key loss are
all YELLOW the moment Chronicle would own them.

### Escalation (YELLOW)

V0 security boundary: **document Option 1** and refuse broad
personal-data capture until the operator’s host encryption is
actually on. Do not implement Option 2–5 unless Ready Room asks.

---

## Program C — Git vs vault

Candidate logical contract (physical backends may change):

```text
PRIVATE CHRONICLE GIT
    small structured records
    source graphs
    manifests / receipts that do not carry source bodies
    later evaluations / derived records
    configuration that is safe to clone

PRIVATE SOURCE VAULT
    exact raw source artifacts
    large or highly private bytes

PUBLIC ENGINE / DOCS
    code and sanitized research
    no private corpus
```

They need not be different directories because this note draws two
boxes. They **must not** be the same ordinary Git history if that
would make forgetting a rewrite-and-GC problem, grow the repo with
binaries, or risk an accidental remote.

**Current practice already matches the contract:** specimen vault
objects live outside git; public repos have machinery and fixtures
only.

**Rejected:** committing raw source bytes into ordinary Chronicle Git
in order to “make them durable.” Durability is the vault’s job (and
later, boring backup tooling). Git clones, remotes, and history are
the wrong forgetting and publication surface.

No YELLOW required to keep this contract. YELLOW if someone proposes
to put vault objects on a Git remote.

---

## What this review does *not* do

- Implement Chronicle-owned encryption (V1 uses OS disk encryption).
- Start a day-view, interpretation, E8, Path, or AMP.
- Authorize live personal export ingest (RED pilot).
- Close privacy-and-forgetting checklist items until the V1 controls
  exist in the engine.

## Approved V1 defaults (Ready Room 2026-08-26)

The three YELLOW forks are no longer stops:

1. **Gate 4:** scope-only forgetting. Implement STOP / WITHHOLD /
   delete-what-we-control. No git-tracked content-hash tombstone. No
   byte-level refuse. No crypto-shred. No vendor-deletion claims.
2. **Security:** private local vault + OS disk encryption. Design
   object custody so a later backend can change without rewriting
   content identity.
3. **Program E:** a narrowly scoped poller/watcher is allowed. Frozen
   v0.1 Activity hooks must not be reused. No generalized daemon first.

## Program D result

SUCCESSIVE-V0 passed (synthetic mutable fixture; not live user data).

## Next

Engine V1 observe path. See the Build Charter envelope.
