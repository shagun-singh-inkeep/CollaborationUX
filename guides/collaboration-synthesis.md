---
title: "Human-to-human collaboration: UX & data-model research"
description: Synthesis of the collaboration product survey (Liveblocks, Figma, Google Docs, Notion, Yjs/Hocuspocus) — extracted presence & comment-thread UX patterns, a proposed ephemeral-vs-persisted collaboration data model, recommended patterns, and open questions for Open Knowledge Phase 2.
type: research
category: research
status: draft
owner: shagun@inkeep.com
created: 2026-07-01
last_verified: 2026-07-01
tags:
  - collaboration
  - ux
  - research
  - presence
  - comments
  - data-model
  - phase-2
---
# Human-to-human collaboration: UX & data-model research

**Status:** draft · **Owner:** shagun@inkeep.com · **Hub:** [[README|Collaboration UX]] · **Feeds:** [[specs/human-to-human-collaboration/spec|Phase 2 spec]] · **Answers open questions** #3 (comment storage), #4 (permissions), #5 (unified identity), #6 (conflict UX) in [[proposals/0001-collaboration-ux|Proposal 0001]]

## Why this doc exists

A research pass on how collaborative docs should behave when **multiple humans** — not just agents — work in the same knowledge base. It exists to de-risk [[specs/human-to-human-collaboration/spec|Phase 2]] before we build the big rocks (comments, view-only, live human presence). Nick named three things specifically: **comment-thread-type things, presence, and metadata related to collaborative docs.** Those map cleanly onto the workstream's [[proposals/0001-collaboration-ux|three pillars]] — Presence, Change legibility, Control — so this survey is organized to feed them.

This doc is the **synthesis**. The per-product studies live in their own docs:

- [[guides/live-blocks|Liveblocks]] — the anchor case study (the collaboration *data model*).
- [[guides/figma|Figma]] — presence as the product.
- [[guides/google-docs|Google Docs]] — comments & suggestions.
- [[guides/notion|Notion]] — block-anchored collaboration.
- [[guides/yjs-baseline|Yjs / Hocuspocus]] — the substrate OK already runs.

## TL;DR — recommendations

1. **Steal&#x20;**[[guides/live-blocks|Liveblocks]]**' layer taxonomy wholesale as our mental model:** *presence (ephemeral) · broadcast (ephemeral) · storage (persisted CRDT) · comments/threads (persisted side store) · notifications (persisted per-user).* OK already has the first three; comments and notifications are the genuine gaps.
2. **Cursors are for humans; diffs are for agents.** Confirmed by every tool surveyed — real-time cursor/selection presence is the right metaphor for char-by-char human typing, and it composes with the agent burst-diff metaphor from [[specs/agent-change-visibility/spec|Phase 1]] in one [[README|presence bar]].
3. **Anchor comments with the editor's relative-position mechanism (Yjs `RelativePosition`), store the thread body + metadata in a side store.** This is the hybrid [[guides/live-blocks|Liveblocks]]/[[guides/notion|Notion]]/[[guides/google-docs|Google Docs]] converged on. It resolves open question #3: neither "fully in-CRDT" nor "fully side-store," but *anchor in-CRDT, payload side-store.*
4. **Presence is ephemeral and must be allowed to be wrong.** Never persist cursor/selection/viewport; let it evaporate on disconnect. Persist only identity, roles, and the durable metadata (owner, last-edited-by, activity state).
5. **View-only is a real product state, not a repo permission** — matches the spec's G4. Minimal viewer/editor split first.

## Product survey — honorable mentions

Beyond the five studies above:

- **Linear / Height** — presence-light but excellent at *activity legibility* (who changed what, compact timelines). Reinforces that OK's [[README|Timeline tab]] is already ahead of the field on the change-legibility pillar.
- **Multiplayer code editors (VS Code Live Share, Cursor)** — follow-the-driver, shared selection; agents as a side pane. OK deliberately differs: the agent is *in* the doc, not a side pane.

## Part 1 — UX patterns extracted

### How people discover others are present

| Pattern | Weight | Use in OK |
| --- | --- | --- |
| Avatar stack ("who's in the room") | light | **Shipping** — the PresenceBar already surfaces human peers (per-doc awareness) alongside agents |
| Live cursor + name label | medium | **Shipping** — humans only; `yCursorPlugin` renders remote carets + name labels in both WYSIWYG and source mode |
| Selection highlight | medium | **Shipping** — humans (same `yCursorPlugin` decorations); agent equivalent is write-flash |
| Typing / "editing now" indicator | light | Optional; ephemeral via awareness or CC1 |
| Active vs idle (greyed avatar) | light | Yes — cheap, high-value |
| Follow / spectate a participant | heavy | Later (G1 optional) — Figma follow-mode |

**Principle:** presence weight should scale with intent. A viewer gets an avatar chip; an active co-editor gets a cursor; someone you choose to follow gets viewport sync. Quiet by default, depth on demand (mirrors the proposal's progressive-disclosure mitigation).

### How edits & conflicts are communicated

- **Live human edits → cursors + the unified&#x20;**[[README|Timeline]], not diffs. Char-by-char typing under a cursor is self-explanatory; the Timeline captures the durable record.
- **Agent edits → burst diffs + summaries** ([[specs/agent-change-visibility/spec|Phase 1]]). The two populations share the presence bar and the timeline but use different change surfaces. *Cursors for humans, diffs for agents.*
- **Two distinct conflict classes must look different (open question #6):**
  - *Layer 1 — live conflict-free merge (CRDT).* No UI needed; convergence is automatic. Don't scare the user with "conflict" language here.
  - *Layer 2 — async git-sync conflict on pull.* Needs a real, first-class resolve UI (the spec calls today's surface "thin"). This is the only place a merge-gate belongs.
- **Suggestions (opt-in) are the one gated affordance** — accept/reject, in contrast to the auto-apply everywhere else.

### How shared context is surfaced

- **Comment threads in a gutter/margin**, anchored to content, with reply/resolve/mention — the single largest missing surface vs every comparable tool.
- **Durable per-doc metadata** (owner, last-edited-by/at, collaborators, activity state) shown in-UI — Nick's "metadata related to collaborative docs."
- **A unified activity feed** — OK's Timeline already does this across humans + agents; Phase 2 refines rather than builds it.
- **Notifications / inbox** for mentions and replies — persisted per-user; the second genuine gap.

## Part 2 — Proposed collaboration data model

The core question (Nick's, and open questions #3/#5): **what is document content vs collaboration state, and what is ephemeral vs persisted?**

### The four-quadrant split

| State | Scope | Persistence | Where it lives in OK | Status |
| --- | --- | --- | --- | --- |
| **Document content** | per-doc | persisted, CRDT-merged | `Y.XmlFragment` + `Y.Text` (the existing editor substrate) | shipped |
| **Presence** — cursor, selection, viewport, focus, `isTyping`, active/idle | per-user, per-doc | **ephemeral** — awareness, dropped on disconnect | Yjs Awareness (`'human'` vs agent discriminator; `agentPresence` map) | shipped for agents; wire for humans |
| **Broadcast signals** — reactions, pings, "look here" | per-user → room | **ephemeral** — fire-and-forget | CC1 push-over-awareness channel | substrate exists; unused for humans |
| **Comment threads** — body, author, `createdAt`, `resolved`, replies, mentions, reactions, **anchor** | per-doc (anchored to a range) | persisted (side store); **anchor via `Y.RelativePosition`** | **gap** — new | build |
| **Notifications** — mention/reply inbox | per-user | persisted per-user | **gap** — new | build |
| **Durable doc metadata** — owner, last-edited-by/at, collaborators, activity state | per-doc | persisted | frontmatter + shadow-repo history / Timeline | mostly shipped |
| **Identity & roles** — display name, avatar, color, viewer/editor role | per-user, per-workspace | persisted | principal identity + writer-ID taxonomy; **roles = gap** | identity shipped; roles gap |

### The comment-anchor decision (resolves open question #3)

**The question.** When someone comments on a span of text, where does the tie between the comment and that text live? A comment really has two parts:

- the **anchor** — *which* text the comment points at, and
- the **payload** — the thread itself: body, author, replies, `resolved` state, mentions.

The spec framed this as a single either/or:

- **All in the CRDT** — conflict-free and always consistent, but heavier, and it bloats the document.
- **All in a side store** — lighter and easy to query, but keeping the anchor pointed at the right text as the document changes is hard.

**The decision: don't choose — split the comment in two.** Every mature tool we surveyed reached the same hybrid — put the **anchor in the CRDT** and the **payload in a side store**, so each half lives where it's naturally strongest.

**Anchor → in the CRDT**, as a `Y.RelativePosition` (or a ProseMirror mark via `y-prosemirror`). Because it rides inside the CRDT, it:

- moves with the text under concurrent edits, conflict-free;
- survives an agent rewriting the whole file;
- degrades gracefully — if its text is deleted it becomes "orphaned / context-lost" ([[guides/google-docs|Google Docs]] behavior) instead of silently pointing at the wrong words.

**Payload → in a side store**, keyed by thread id. Kept out of the document, the thread body is:

- lighter — the document Y.Doc doesn't grow with every comment;
- easy to query and to drive notifications from.

This is exactly [[guides/live-blocks|Liveblocks]]' managed-thread-store model.

**One sub-decision still open — how precise the anchor is:**

- a **character range** is the most precise (it highlights the exact words) but fragile under heavy editing;
- a **block-level anchor** ([[guides/notion|Notion]]-style) is coarser but far more stable.

Given OK's ProseMirror block model, supporting *either a block or a range* is a reasonable v1. Pick the default with a stability test — the spec's test plan already calls for "comment stays attached under concurrent edits."

**Bottom line:** anchor in-CRDT + payload side-store keeps the document CRDT clean while still getting conflict-stable anchoring — the best of both options the spec posed.

### Ephemeral vs persisted —==&#x20;the rule&#x20;==

- **Ephemeral (never persist, allow to be wrong):** cursor, selection, viewport, focus, typing, active/idle, ad-hoc reactions/pings. Losing these on disconnect is *correct*.
- **Persisted (durable, merged or queryable):** document content, comment threads + anchors, notifications, doc metadata (owner/last-edited/activity), identity, roles.
- **The trap:** persisting presence (stale ghost cursors) or putting comment bodies inline in the text CRDT (bloat + fragile). Keep the boundary sharp.

### Per-user / per-doc / per-workspace

- **Per-user:** identity (name, avatar, color), role, notification inbox, and *their* ephemeral presence.
- **Per-doc:** content, comment threads, durable metadata (owner, last-edited, activity state), the set of currently-present participants.
- **Per-workspace:** membership, roles/groups, and (open question #4) where permission enforcement lives — federated store vs git-delegated.

## Part 3 — Recommended patterns for Open Knowledge

1. **Human presence is already shipping** — the PresenceBar surfaces human peers and `y-prosemirror`'s `yCursorPlugin` renders live cursors + selection in both editor modes. This one's essentially done; remaining polish is active/idle states. Reuse, don't rebuild (see [[guides/yjs-baseline|Yjs baseline]]).
2. **Build comments as the flagship Phase 2 surface** — gutter threads, reply/resolve/`@`-mention, **anchor in-CRDT + payload side-store**. This is the biggest gap and the highest user value.
3. **Add a per-user notification inbox** for mentions/replies — the second gap; small but expected.
4. **Keep one identity model** across humans + agents (writer-ID taxonomy) so the Timeline and comments read coherently — satisfies open question #5.
5. **Make view-only a real product state** (editor renders read-only; all write surfaces refuse) — minimal viewer/editor split first, RBAC later. Note the [[guides/live-blocks|Liveblocks]] nuance: a viewer can still *broadcast presence*, so you can see their cursor without granting edit.
6. **Give Layer-2 git conflicts a first-class resolve UI**, visually distinct from conflict-free live edits — resolves open question #6.
7. **Progressive disclosure throughout** — quiet by default, depth on demand, so presence + comments + diffs + timeline don't clutter the editor (the proposal's stated drawback).

## Part 4 — Open questions & tradeoffs

- **Anchor granularity: range vs block.** Range = precise but fragile; block = stable but coarse. Needs a concurrent-edit stability test before committing (feeds the spec test plan).
- **Comment store location.** Side store where? The federated backend (Miles' track), a local `.ok/` store, or git-tracked? Ties to open question #4 and the federated-sync dependency.
- **Permission enforcement point (open #4).** Federated store owns roles vs git-delegated with a thin UI veneer. G4 blocks on this.
- **Suggestion model scope.** Full [[guides/google-docs|Google Docs]] suggestion mode vs a minimal propose/accept. Value vs build cost.
- **Invite & identity flow.** Link / account / GitHub identity — how a person joins a shared session. Depends on the federated backend's auth. Does view-only imply a **hosted read view** (a shareable link, no OK install), bridging from today's `share_link` (pull-into-your-own-install) model?
- **Notification transport.** In-app inbox only, or email/OS notifications too? Cadence/noise control overlaps open question #1.
- **Presence at scale.** Awareness broadcasts to all peers; large rooms need throttling/cursor-culling. Not urgent, but a known Yjs sharp edge.

## Links

- Hub: [[README|Collaboration UX]]
- North star: [[proposals/0001-collaboration-ux|Proposal 0001 — Collaboration UX]] (open questions #3–#6 are answered/advanced here)
- Phase 2 spec: [[specs/human-to-human-collaboration/spec|Human-to-human collaboration]] · [[specs/human-to-human-collaboration/tasks|tasks]]
- Phase 1 (the legibility model this reuses): [[specs/agent-change-visibility/spec|Agent change visibility]]
- Product studies: [[guides/live-blocks|Liveblocks]] · [[guides/figma|Figma]] · [[guides/google-docs|Google Docs]] · [[guides/notion|Notion]] · [[guides/yjs-baseline|Yjs/Hocuspocus baseline]]
