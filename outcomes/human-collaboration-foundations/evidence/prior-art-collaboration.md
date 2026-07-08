---
title: Collaboration-experience prior art — Notion, Google Docs, Figma, Obsidian Relay, Linear, AI editors (channel 3)
description: Synthesis of how comparable products handle the five collaboration-UX axes (identity/mentions, comments, view-only, notifications, AI-change indicators), classed into table stakes / differentiators / deliberately-avoid. Includes the Notion Updates-feed deep-linking addendum.
tags:
  - evidence
  - collaboration
  - phase-2
  - synthesis
  - prior-art
type: synthesis
created: 2026-07-08
method: WebSearch probes (2026-07-08) + one internal cross-product report (agent-follow-and-edit-visibility-ux) for AI-editor facts
confidence_note: web-sourced claims are SUPPORTED unless marked CONFIRMED by vendor docs; product behavior can drift.
---
# Prior art: how comparable products handle the five collaboration-UX axes

Related captured sources in this KB: [Notion sharing & permissions](../../../external-sources/notion-sharing-permissions.md) · [Google Docs sharing roles](../../../external-sources/google-docs-sharing-roles.md) · [Figma share & permissions](../../../external-sources/figma-share-permissions.md) · [Liveblocks room permissions](../../../external-sources/liveblocks-room-permissions.md) · [GitHub repository roles](../../../external-sources/github-repository-roles.md) · [GitHub collaborators & contributors APIs](../../../external-sources/github-list-collaborators-contributors.md).

## 1. Notion (workspace-member model, closest product analog)

- **(a) Identity/mentions — SUPPORTED:** four permission levels (Full access / Can edit / Can comment / Can view). Two population classes: *members* (workspace-wide) and *guests* (per-page invitees by email). @-mention directory is access-scoped: you can only mention people who share access to the page/teamspace — mentionability is derived from permissions, not a global directory. See [Notion sharing & permissions (captured)](../../../external-sources/notion-sharing-permissions.md); also notion.com/help/add-members-admins-guests-and-groups.
- **(b) Comments — SUPPORTED:** inline anchored comments + page-level comments; threads with resolution; "Can comment" exists as an explicit middle tier between view and edit.
- **(c) View-only — SUPPORTED:** "Can view" exists and excludes commenting; teamspace-level defaults. Guest model is the growth loop (guests later convert to members).
- **(d) Notifications — SUPPORTED:** in-app inbox (Updates), triggered by mentions, comments, page shares; email/slack mirrors. Subscription is implicit via involvement.
- **(e) AI attribution — SUPPORTED:** page history "displays who made each change"; Custom Agents get *full audit trails* ("every agent run is logged... what triggered it, what it did, and why"; enterprise audit log includes agent activity + who triggered it). Notion treats the agent as an actor in history + a dedicated agent-activity log, not as inline colored text. Sources: notion.com/help/notion-agent, notion.com/help/custom-agents.
- **Not chosen / why (INFERRED):** no per-character authorship coloring (history-diff granularity instead); no anonymous collaborators — identity is account-mandatory.
- **Context delta vs OK:** fully hosted, account-first; OK is local-first with no accounts and agents as first-class MCP editors.

## 2. Google Docs (the anchoring + review-gate reference)

- **(b) Comments/anchoring — CONFIRMED (vendor issue tracker + support threads):** comments anchor to text ranges; when the anchored text is deleted the comment survives as **"Original content deleted"** (orphaned, unanchored). Comments are NOT part of version history — restoring an old version does not restore deleted comments (long-standing user complaints). Sources: github.com/googleworkspace/cli/issues/169, support.google.com/docs threads 137281113, 180647431.
- **(a) Identity — SUPPORTED:** Google-account model + anonymous-animal viewers for link-shared docs (the "Anonymous Kangaroo" pattern — precedent for OK's Adjective-Animal fallback).
- **(c) View-only — SUPPORTED:** Viewer / Commenter / Editor roles; the canonical three-tier. See [Google Docs sharing roles (captured)](../../../external-sources/google-docs-sharing-roles.md).
- **(d) Notifications — SUPPORTED:** email-centric; per-doc comment notification settings (all / involving me / none).
- **(e) AI/agent-change indicators — SUPPORTED:** the relevant primitive is **Suggesting mode** (tracked changes with accept/reject) — a human review-gate that products increasingly reuse for AI edits; version history shows per-editor colored diffs and "named versions."
- **Not chosen / why (INFERRED):** OT lossless merge lets Docs make version history manual-access-only and keep collaboration silent; no artifact-based recovery needed.

## 3. Figma (presence-first; comment-is-the-only-write for viewers)

- **(a)/(c) — SUPPORTED:** seat/role model; the free View seat can view **and comment** everywhere — commenting is deliberately not gated to paid/edit seats. Notable gap users complain about: no "view without commenting" option (forum threads requesting comment-free view-only). See [Figma share & permissions (captured)](../../../external-sources/figma-share-permissions.md); forum.figma.com threads 32034, 15813, 42122.
- **(b) — SUPPORTED:** pin-style comments anchored to canvas coordinates/objects; threads + resolve; comments require a (free) account.
- **(d) — SUPPORTED:** email + in-app notification settings per file (all / mentions-only / none).
- **(e) — SUPPORTED (internal report cross-check):** Figma's merge-review pattern (branch Review/Dismiss dialog) is ceremony-scoped, not live-typing-scoped; live multiplayer has no per-change attribution beyond presence/cursors.
- **Context delta vs OK:** presence (avatars, cursors, observation mode) is the identity surface; files are hosted; "who's here" matters more than "who wrote this byte."

## 4. Obsidian + Relay plugin (the local-first CRDT analog)

- **(a) — SUPPORTED:** Relay (system3) adds Yjs-CRDT multiplayer to Obsidian via *shared folders* on a relay server; collaborators join via **share keys** (invite tokens), identity via login to the relay service (proprietary permissions/billing server); business tier adds "advanced permissions." Local-first: edits tracked locally, merged on reconnect. Sources: relay.md, github.com/No-Instructions/Relay, forum.obsidian.md thread 87170.
- **(b) — NOT FOUND:** no anchored comment/thread system in Relay or core Obsidian (comments are `%%markdown%%` literals — same as OK today).
- **(c) — SUPPORTED:** folder-granular sharing is the access model; per-folder membership rather than per-doc roles.
- **(d) — NOT FOUND:** no notification inbox; Obsidian Sync's silent merges without any warning generated a 4-year-open forum complaint (cross-ref internal silent-loss report).
- **(e) — n/a** (no first-class agents).
- **Context delta vs OK:** closest architecture (local-first markdown + CRDT + optional relay). Relay's choices — share-key onboarding, folder-scoped membership, proprietary auth server bolted onto a local-first core — are the nearest existence proof for OK's federated direction.

## 5. Linear (the notification-model reference)

- **(d) — CONFIRMED (vendor docs):** single in-app **Inbox**; notifications flow from *subscriptions*; you are auto-subscribed by creating, being assigned, or being @-mentioned (mention in a comment subscribes you to the thread, not the whole issue). Every other channel (email, Slack, push) links back to the Inbox item. Per-trigger opt-outs. Sources: linear.app/docs/notifications, linear.app/docs/inbox.
- **(a) — SUPPORTED:** workspace-member directory; mentions of users and teams.
- **(b) — SUPPORTED:** issue comment threads (document-level, not text-anchored); emoji reactions; resolve-by-convention.
- **(c) — SUPPORTED:** guest role restricted to specific teams; view-only largely handled by workspace membership rather than per-doc ACLs.
- **Deliberate choice:** implicit subscription + one inbox — no per-doc notification settings sprawl.

## 6. AI-native editors (Cursor / Zed / GitHub Copilot / Claude Code) — agent-change indicators

- **Pending-review diffs — SUPPORTED:** Cursor's core pattern is inline red/green diffs with per-chunk and per-file Accept/Reject; recent regressions where agents auto-applied without review produced immediate loud user backlash ("you're throwing away your best UX advantage") — evidence that a *review gate* is perceived as the product's value, and that removing it is felt as a trust breach. Sources: cursor.com/docs/agent/review, forum.cursor.com threads 152581, 160856, 146198.
- **Commit-trailer attribution — SUPPORTED:** VS Code/Copilot auto-append `Co-authored-by: Copilot` was controversial for being silent-by-default; default flipped to off — industry shift to explicit opt-in AI provenance; trailer granularity criticized (can't distinguish inline completion from agentic contribution).
- **Session/audit views — SUPPORTED (internal report):** Devin-style session replay / Claude Code transcript as the "what did the agent do" surface; GitHub Copilot agent PRs use draft-PR-as-review-boundary.
- **Context delta vs OK:** code editors get a free review boundary (git commit / PR). OK's agents write *live into the shared CRDT* — there is no natural staging area, which is exactly why OK built flash + presence + activity panel + timeline instead of accept/reject gates.

---

# Synthesis: table stakes / differentiators / deliberately avoid

## Table stakes (every comparable product has these; users will assume them)

1. **Identity for every visible actor** — name + avatar/color, stable across sessions (Notion/Docs/Figma/Linear all account-backed; even Relay requires login). Anonymous-animal identities are acceptable only for *view-scoped* drop-ins (Docs).
2. **Access-scoped @-mention directory** — mention whoever shares access to the doc/space; mention ⇒ notification (Notion, Linear, Figma, Docs).
3. **Anchored comments with threads + resolve**, with a defined orphan behavior when anchored text changes (Docs: "original content deleted"; comments decoupled from version restore).
4. **Attribution in version history** — who changed what, when, including AI actors as named authors (Notion agents, git co-author trailers).
5. **A notification surface with implicit subscription** (mentioned/involved ⇒ notified) and per-trigger opt-out (Linear's inbox is the cleanest reference).

## Differentiators (open territory OK could own)

1. **First-class agent actors in the same collaboration fabric as humans** — presence, per-write summaries, per-agent activity panel, timeline attribution. No shipping product unifies human + agent identity in one presence/history model.
2. **Git-derived collaborator identity** — recovering real teammate authorship from the git substrate without accounts (upstream-author attribution). No peer does account-less collaborator attribution.
3. **Artifact-first "notification" philosophy** — durable, discoverable artifacts (checkpoints, timeline rows, summaries) over interruptive toasts (validated by internal silent-loss report + Obsidian counter-example).
4. **Agent review-boundary innovation** — OK agents write live (no PR gate); a lightweight follow/rewind/undo-per-agent affordance set (pause-the-agent, per-agent undo, burst scrubbing) has no prior art.
5. **Comment threads that agents can read/write** — if comments become MCP-visible objects, agents can participate in review conversations; no markdown-native product has this.

## Deliberately avoid (chosen-against patterns with documented failure modes)

1. **Silent AI auto-apply without any visibility** — Cursor's regression backlash; Copilot silent co-author controversy. Visibility (not necessarily blocking review) must be default-on.
2. **Blocking mid-typing conflict/merge dialogs and loss-toasts** — anti-pattern per internal silent-loss report; no production editor does it.
3. **Interruptive notification sprawl** (per-event emails, unbatched toasts) — Linear's single-inbox + Figma's mentions-only default are the corrective.
4. **Comment-permission maximalism at v1** — Figma's inability to offer "view without comment" and Notion's four-tier matrix both generate support burden; a middle "can comment" tier is table stakes *eventually* but tiering beyond viewer/commenter/editor is not.
5. **Cross-user identifiers embedded in share URLs / sender attribution on public surfaces** — already a locked OK decision (sharing spec D21/D31, privacy-driven).
6. **Per-character authorship coloring as the primary attribution surface** — nobody ships it for text at scale; per-change/per-commit diff granularity is the norm (Docs colored version diffs, Cursor chunk diffs, OK bursts).

## Addendum (2026-07-08): Notion Updates-feed mechanics (user-supplied screenshot + product knowledge)

Prompted by Shagun's screenshot of a Notion page's Updates sidebar; informs O1's timeline/navigation ACs.

- **SUPPORTED (screenshot):** per-page reverse-chron feed; edit entries batched per editor per session ("Nick Gomez edited <page>, May 4" covering many edits); inline block-based snippet diffs with blue highlight on changed text, collapsed behind "View N more"; permission changes ("updated permission… Full access") ride the same feed → one durable mixed event stream per page.
- **CONFIRMED (earlier sweep):** agents appear as named actors in the same history + dedicated audit trails (notion.com/help/custom-agents).
- **CONFIRMED (user screenshot, 2026-07-08 — CORRECTS the earlier claim below):** feed diff snippets DO deep-link to the changed content. Mechanism visible in the screenshot's URL: `.../<Page-Title>-<page-uuid>#<block-uuid>` — clicking an edit snippet navigates to the page anchored at the changed block's stable UUID, and the main pane scrolls to it. Notion gets this "for free" from its block model: every block is an entity with a durable UUID; the edit event stores block refs plus before/after content (which renders the added-text highlight in the snippet); the link is an ID anchor, not a text offset, so it survives later edits as long as the block exists. Known failure mode: if the block was deleted, the anchor silently lands on the page with no "content deleted" affordance.
- ~~SUPPORTED: feed entries do not deep-link~~ — retracted per the screenshot above. What remains SUPPORTED (long-standing complaints): page *history* (version/restore surface) is snapshot-based with no inter-version diff highlighting — the deep-linking lives in the Updates feed only.
- **Implication for O1 (revised):** Notion is direct prior art for the jump-to-edit AC — it proves the exact interaction at scale (durable feed entry → inline snippet diff → click → land on the content in the live doc). The load-bearing delta: Notion anchors by stable block UUID; OK's markdown-CRDT model has no block IDs, so /spec must derive equivalent durable anchors — CRDT relative positions are the natural analog (same "stable identity, not text offset" property). This de-risks the O1 concept while keeping OK's re-anchoring assumption as the engineering question. OK can still beat Notion's gaps: explicit deleted-content affordance instead of a silent no-op, and Docs-style prev/next stepping through a session's changes, which Notion lacks.

## Addendum (2026-07-08): Liveblocks product-surface sweep (presence, metadata, agents)

Prompted by Shagun's question "any outcomes derivable from presence, metadata, and how platforms like Liveblocks handle collaborative docs?" — web-verified 2026-07-08.

- **CONFIRMED (liveblocks.io + docs):** product surface = Presence (live cursors, avatar stacks, selection indicators), Comments (anchored threads, replies, mentions, emoji reactions, resolve/unresolve), Notifications (inbox component + email + webhooks), and — since 3.0 — **AI Copilots**: "AI agents that participate in collaborative sessions alongside human users — read and write shared documents, respond to comments, and show presence just like a real user."
- **COMPETITIVE IMPLICATION (O1/O8):** the "no shipping product unifies human + agent collaboration fabric" differentiator claim is eroding — Liveblocks now sells agent-participation primitives *as infrastructure*. OK's remaining edge: local-first + git substrate; MCP-native (any agent/harness the user already runs, not one built-in copilot); attribution depth (durable git-backed provenance); agent-directed threads on plain markdown. Urgency signal for the local track.
- **SUPPORTED (existing capture: [Liveblocks room permissions](../../../external-sources/liveblocks-room-permissions.md)):** the permission grammar separates `room:read` / `room:write` / `room:presence:write` — **presence is its own permission axis**: a read-only member can still broadcast presence. Feeds O7's role×presence rules and O6 tier design.
- **INFERRED (API design; verify at /spec):** user identity enters Liveblocks as host-supplied resolver callbacks (resolve users by id / suggest mention candidates, with names + avatars) rather than platform-owned data — the directory contract XQ2 must confirm with the member-management workstream is essentially that resolver interface plus access scoping.
- **DERIVED OUTCOME → O9 (sync-state legibility):** collab infrastructure treats connection/sync state as a first-class primitive; in a multiplayer workspace, silent staleness is the new silent loss — a dropped connection quietly turns a shared doc into a private fork. Generalizes O6's offline-viewer divergence state to every role; consistent with the house silent-loss philosophy.
