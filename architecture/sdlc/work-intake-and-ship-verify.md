# Work intake, ship bundles, and stakeholder verify

Operator process that sits **in front of** the SDLC engine. The engine
speaks GitHub issue refs (`sdlc-workflow drop`, `run`). This document is
how those issues get created, how same-sitting work ships together, and
how a stakeholder who does not use GitHub checks a sandbox drop.

Policy (which hosts, which Slack list, what must never appear on a verify
row) stays in the consumer workspace — [ADR-0009](../ADR-0009-platform-boundary-mechanism-vs-policy.md).
Mechanism (layers, routes, bundle grain, publish-not-poll) is here.

Companion templates: `team-setup` rules `work-intake` and
`stakeholder-verify-watch`, plus the Slack list helper under
`stakeholder-verify-watch/scripts/`. Drop grain: [PRD-0026](../../product/PRD-0026-drop-mode-sdlc.md).

## 1. Layers (do not collapse them)

| Layer | Job | System of record |
| --- | --- | --- |
| **Intake** | Raw ask | Transcript, Slack, a tracker, a prompt |
| **Engineering ledger** | Is this still open? | GitHub Issues (`direct` / `bug-spec` / `plan`) |
| **Product contract** | What to build and what “done” means | PRD in the docs repo that owns the product |
| **Durable decision** | Must still bind in a year | ADR |
| **Execution envelope** | Paths, gates, tests | Spec (`sdlc-workflow`) when the route is `run` |
| **Delivered** | What we intended to ship | `docs/releases/YYYY-MM-DD.md` in the app repo |
| **Stakeholder check-off** | Did a non-GitHub stakeholder smoke this? | External ledger (Slack list by default) |
| **Engineering memory** | Why we did it and how we got here | Chronicle (observational memory, not a backlog or work ledger) |

Intake is **not** the backlog. **Catalog** it: promote each ask to an
issue (or a PR that will close the same sitting). If it is not an issue,
it is not committed work.

These layers are **different operational systems of record**. Chronicle
does not replace any of them. In this SDLC context it preserves enough
relationship among them that later someone can ask “why did we ship
this?” and traverse intake → issue → decision → implementation →
sandbox release → verification. Chronicle is not where work waits.
Chronicle is where the path survives — whether captured while it
happened or reconstructed later.

A domain-expert **Feedback** tracker (or similar inbox) stays an inbox.
It is a bad second backlog. Point stakeholders at issue URLs or the
verify list — do not re-type the same bullets into chat and GitHub
forever.

## 2. Cataloging (route every issue)

Every committed item gets a route:

| Route | Use | Next |
| --- | --- | --- |
| `direct` | Small, obvious, safe | One drop / PR. `Closes #N`. |
| `bug-spec` | Real bug with blast radius | `/write-bug-spec` → same run machine, no PRD |
| `plan` | Feature-sized | Plan → PRD → Accepted → `decompose` → spec → `run` |

Do **not** `sdlc-workflow decompose` until the PRD is **Accepted**.
Do **not** write a PRD for a same-day **bundle** (below).

**PRD vs ADR:** a PRD is a thing to build. An ADR is a decision that
should not be relitigated. Product rules live in the PRD, not an ADR.

When the operator pasted a Slack permalink as the request, record it on
the GitHub issue (Source / body). After the fix is **deployed to SB**
(sandbox — stakeholders say Sandbox / SB, not “dev”), reply **in that
same thread**. No `@channel`. No regulated or sensitive data. Not on
push or CI green — only `deploy_green` for the SHA that contains the
fix. See `deploy-verify-watch`.

## 3. Bundle (same-sitting ship)

A **bundle** is a named set of cataloged issues that ship together in
one sitting: one drop, one PR, one sandbox deploy, one verify list.
PRD-0026’s drop is the engine grain; the bundle is the operator
decision to **group** those issues instead of opening a PRD or a
per-item drop.

Use a bundle when:

- The items are already planned (cataloged issues with `direct` or
  obvious `Done-when`).
- They should land on the same sandbox host(s) the same day.
- None of them needs a year-binding contract of its own.

Do **not** bundle a feature that still needs an Accepted PRD into a
same-day ship. Do **not** `decompose` a bundle into per-task PRs.

## 4. Delivered items and the changelog

User-facing work has **two** release surfaces. Do not skip the sandbox
one.

1. **PR body** `## Release notes` — one stakeholder line. Feeds the
   GitHub Release **after** promote-to-prod (too late for the sandbox
   pass).
2. **Dated sandbox note** `docs/releases/YYYY-MM-DD.md` —
   **Delivered**, **Not verified**, **Verified**, **Out of this ship**.
   Append to today’s file when it is the same bundle. Index it in
   `docs/releases/README.md`.

Convention inside the dated file:

- Add a checkbox under **Not verified** when the item lands on the
  sandbox (or is deploying and will be there when smoke starts).
- The external ledger’s Status is whether the stakeholder smoked it.
  Do **not** live-sync git checkboxes from a laptop watcher. Promote
  snapshots Verified into this file — never delete a line.
- No regulated identifiers, customer names, or production dumps.

## 5. Stakeholder verification ledger (no GitHub required)

In this SDLC context, Chronicle is engineering memory: it preserves
the path of work and decisions; it is not the stakeholder verification
ledger. Stakeholders who do not use GitHub need an operational
check-off surface they already live in. Default mechanism: a **Slack
List**.

| List | Job |
| --- | --- |
| **Feedback** (or equivalent inbox) | Asks. Promote out to Issues. |
| **Sandbox verify** | Check-off for a named bundle. Status is truth for “did they smoke this?” |

Do **not** mix the two lists.

### 5.1 List shape

Columns (no regulated or sensitive data):

| Column | Values |
| --- | --- |
| Item | Smoke line (same text as the dated release note) |
| Host | Consumer sandbox host names, plus `prod` |
| Status | `Not verified` / `Verified` / `Failed` |
| Ship | Issue number or drop date (`#123` / `2026-08-13`) |
| Notes | Optional fail note |

List id and column ids live in workspace Slack env **and** the product
repo’s Actions variables (`VERIFY_SLACK_LIST_ID`, `VERIFY_COL_*`).
Create the list **once**; agents upsert rows. Column ids are not
secrets.

### 5.2 Loop (hosted, not a laptop)

```
sandbox deploy green
  → update docs/releases (Not verified)
  → upsert Slack Sandbox verify rows
  → notify the configured support channel with the list URL and new
    smoke lines (`VERIFY_NOTIFY_CHANNEL_ID`)
stakeholder sets Status = Verified
  → Slack is the answer; git does not change
stakeholder sets Status = Failed
  → hosted Action comments the Ship issue (deduped)
  → agent fixes, redeploys, republishes the row as Not verified
Promote to prod
  → snapshot Slack Verified into docs/releases
  → upsert prod host rows on the same list
  → stakeholder re-smokes prod
```

Failed: fix, push, republish the row as Not verified. Do not promote.

Do **not** poll Slack from a laptop. Slack Lists have no events API;
the hosted Action is the Failed bridge. Verified does not need a
bridge until promote.

Verified on sandbox is **not** Approve-to-merge. Approve remains the
GitHub proceed signal (`pr-approve-watch`). Promote to prod after the
stakeholder signs off the bundle on Slack, then re-smoke **prod** rows
on the same list.

### 5.3 What the stakeholder does

When new rows are published, the notify channel gets the list URL and
the new smoke lines. Open the list. Work top to bottom. Set
**Verified** or **Failed**. They do not need to message the operator,
open GitHub, or edit Markdown.

## 6. Agent commands

- Cataloging a transcript / Slack dump / prompt: follow this document;
  file issues as Addi; Draft PRDs only for `plan` items.
- Shipping a user-facing PR: fill `## Release notes` **and** the dated
  `docs/releases` file.
- After a live sandbox deploy: `/watch-stakeholder-verify` (**publish
  only** — do not arm a local Slack poller). Pair with
  `/watch-deploy-verify` and `/watch-pr-approve`. Failed rows land as
  comments on the Ship issue from the hosted **Sandbox verify** Action.

## 7. Platform boundary

| Mechanism (this doc + `team-setup`) | Policy (consumer) |
| --- | --- |
| Issue routes, bundle grain, dated `docs/releases/` | Which product repos write release notes |
| Slack list upsert / status / failed-notify / snapshot | List id, column ids, notify channel, host names |
| “Do not poll from a laptop” | Which Action repo hosts the Failed bridge |
| Thread reply on `deploy_green` when a Slack permalink was the ask | What “SB” hosts are called in that workspace |
| No regulated data on verify rows | What counts as regulated / forbidden |

The engine does not key on a consumer name, path, or hostname.
`VERIFY_SANDBOX_HOSTS` and `VERIFY_NOTIFY_CHANNEL_ID` are the seams.
