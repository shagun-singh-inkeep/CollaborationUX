---
title: "Human-to-human collaboration: UX & data-model research"
description: Survey of Liveblocks, Figma, Google Docs, Notion and the Yjs/Hocuspocus baseline — extracting presence, comment-thread, and shared-state UX patterns plus a proposed ephemeral-vs-persisted collaboration data model for Open Knowledge Phase 2.
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
  - liveblocks
  - presence
  - comments
  - data-model
  - phase-2
---
# Human-to-human collaboration: UX & data-model research

**Status:** draft · **Owner:** shagun@inkeep.com · **Hub:** [[README|Collaboration UX]] · **Feeds:** [[specs/human-to-human-collaboration/spec|Phase 2 spec]] · **Answers open questions** #3 (comment storage), #4 (permissions), #5 (unified identity), #6 (conflict UX) in [[proposals/0001-collaboration-ux|Proposal 0001]]

## Why this doc exists

A research pass on how collaborative docs should behave when **multiple humans** — not just agents — work in the same knowledge base. It exists to de-risk [[specs/human-to-human-collaboration/spec|Phase 2]] before we build the big rocks (comments, view-only, live human presence). Nick named three things specifically: **comment-thread-type things, presence, and metadata related to collaborative docs.** Those map cleanly onto the workstream's [[proposals/0001-collaboration-ux|three pillars]] — Presence, Change legibility, Control — so this survey is organized to feed them.

The deepest case study is **Liveblocks** (this doc's namesake), because its product *is* the collaboration data model — it forces the exact content-vs-collaboration-state split we need to make. The other products are read for their UX surfaces.

## TL;DR — recommendations

1. **Steal Liveblocks' layer taxonomy wholesale as our mental model:** *presence (ephemeral) · broadcast (ephemeral) · storage (persisted CRDT) · comments/threads (persisted side store) · notifications (persisted per-user).* OK already has the first three; comments and notifications are the genuine gaps.
2. **Cursors are for humans; diffs are for agents.** Confirmed by every tool surveyed — real-time cursor/selection presence is the right metaphor for char-by-char human typing, and it composes with the agent burst-diff metaphor from [[specs/agent-change-visibility/spec|Phase 1]] in one [[README|presence bar]].
3. **Anchor comments with the editor's relative-position mechanism (Yjs `RelativePosition`), store the thread body + metadata in a side store.** This is the hybrid Liveblocks/Notion/Google-Docs converged on. It resolves open question #3: neither "fully in-CRDT" nor "fully side-store," but *anchor in-CRDT, payload side-store.*
4. **Presence is ephemeral and must be allowed to be wrong.** Never persist cursor/selection/viewport; let it evaporate on disconnect. Persist only identity, roles, and the durable metadata (owner, last-edited-by, activity state).
5. **View-only is a real product state, not a repo permission** — matches the spec's G4. Minimal viewer/editor split first.

## Part 1 — Product survey

### Liveblocks (the anchor case study)

Liveblocks is collaboration *infrastructure* — its entire API is a taxonomy of "what kind of shared state is this," which is exactly the question Phase 2 has to answer. Unit of collaboration is a **Room**; participants join a room and get five distinct state layers, each with different persistence and conflict semantics:

| Liveblocks layer | Persistence | Conflict model | Examples |
| --- | --- | --- | --- |
| **Presence** | Ephemeral — in-memory, dropped on disconnect | Last-write-wins per client, no merge | cursor position, selection range, `isTyping`, current focus |
| **Broadcast events** | Ephemeral — fire-and-forget, not stored | None (one-shot) | emoji reactions, "pinged you", toasts |
| **Storage** | Persisted, server-authoritative | Conflict-free (their own `LiveObject`/`LiveList`/`LiveMap` CRDT, **or** a Yjs doc) | the document / canvas / app data |
| **Comments / Threads** | Persisted in a managed side store | Thread-level, queryable | comment body, author, `resolved`, replies, `@`-mentions, reactions, `metadata` (the anchor) |
| **Notifications** | Persisted per-user | — | inbox, mention alerts, thread-reply subscriptions |

What matters for us:

- **The presence vs storage split is the load-bearing idea.** Presence is deliberately lossy and cheap; storage is durable and merged. Trying to make one thing do both is the classic mistake. OK already respects this: **Yjs Awareness = presence, Y.Doc = storage.**
- **Comments are NOT in the document CRDT.** Liveblocks keeps thread bodies/metadata in a separate managed store and anchors them into the text via the editor binding (Lexical / Tiptap / BlockNote plugins) using the editor's relative positions. So the *anchor* rides the CRDT; the *payload* does not. This is the single most important finding for open question #3.
- **Thread `metadata` is an open key/value bag** — teams stash the anchor (quote text, block id, or x/y for canvas), `resolved`, priority, etc. Flexible metadata on the thread is how they avoid a rigid comment schema.
- **Identity is resolved, not stored.** Liveblocks holds only a stable `userId`; the app supplies `resolveUsers` / `resolveMentionSuggestions` to hydrate names/avatars. OK's analog already exists: the **principal identity** (real git name/email) and the [[proposals/0001-collaboration-ux|writer-ID taxonomy]] (`agent-*`, `principal-*`, `file-system`, `git-upstream`).
- **Permissions are per-room, coarse:** `room:write`, `room:read`, `room:presence:write`, granted via access/ID tokens with `usersAccesses` / `groupsAccesses`. Notable: **read-only clients can still broadcast presence** (`presence:write` without `write`) — you can *see* a viewer's cursor even though they can't edit. Directly relevant to OK's G4.

### Figma — presence as the product

- **Buttery multiplayer cursors** with name labels and per-user color; the gold standard for "who is here, where are they looking."
- **Follow mode / observation mode:** click an avatar to lock your viewport to theirs. Spectating is a first-class, named state — the spec's optional viewport-follow (G1) is this.
- **Avatar stack** in the top bar = the canonical "who's in the room" surface. Grey/inactive vs active. This is the [[README|PresenceBar]] pattern.
- Ephemeral spatial presence (cursor x/y, current selection, viewport) is never persisted — pure awareness.

### Google Docs — comments & suggestions

- **Comment threads anchored to a text range**, shown in a right margin/gutter; reply, resolve, `@`-mention (which emails/notifies). The resolved thread disappears from the doc but is retrievable — a persisted side store, not inline text.
- **Anchoring survives edits** via internal anchor/named-range IDs; when the anchored text is deleted the comment goes "orphaned"/context-lost rather than vanishing. Good model for our degrade-gracefully behavior.
- **Suggestion mode:** edits render as tracked insert/delete proposals with per-suggestion accept/reject. This is the heavyweight end of the suggestion open question.
- Presence is lighter than Figma: colored **caret + name flag** and avatar chips, no follow-mode by default.

### Notion — block-anchored collaboration

- **Comments anchor to a block (or an inline range within a block)**, not a character offset. Coarser anchor = more stable under edits, at the cost of precision. A pragmatic middle ground worth considering given OK's block-ish ProseMirror model.
- Presence = avatar chips + lightweight "editing" cursors; not the Figma spectacle.
- **Per-page metadata** (last-edited-by/at, owner, properties) is surfaced in the UI as durable collaboration state — exactly the "metadata related to collaborative docs" Nick asked about.

### The Yjs / Hocuspocus baseline (what OK already runs)

OK is built on the same substrate these tools use under the hood, so much of this is *wiring, not inventing* (echoing the spec's "shipped vs gap"):

- **Awareness protocol = presence.** Ephemeral per-`clientID` state (cursor, selection, user identity/color), broadcast to peers, auto-cleared on disconnect. OK already carries a `'human'` vs agent discriminator here, and an `agentPresence` map for agents.
- **`y-prosemirror`** ships remote-cursor and selection **decorations** out of the box — live human cursors are largely a *turn-it-on* once a reachable shared server exists.
- **`Y.RelativePosition`** — positions that survive concurrent edits. This is the primitive for conflict-stable comment anchors (see the data model below).
- **CC1 push-over-awareness** (OK's existing pure-signal broadcast channel) is the local analog of Liveblocks broadcast events — a place to hang ephemeral "X pinged you"/reaction signals without touching the doc CRDT.

**Implication:** OK's substrate already provides Liveblocks' presence + broadcast + storage layers. Phase 2's net-new build is concentrated in **comments/threads and notifications**, plus making live human edits reachable (the federated-sync dependency).

### Honorable mentions

- **Linear / Height** — presence-light but excellent at *activity legibility* (who changed what, compact timelines). Reinforces that OK's [[README|Timeline tab]] is already ahead of the field on the change-legibility pillar.
- **Multiplayer code editors (VS Code Live Share, Cursor)** — follow-the-driver, shared selection; agents as a side pane. OK deliberately differs: the agent is *in* the doc, not a side pane.

## Part 2 — UX patterns extracted

### How people discover others are present

| Pattern | Weight | Use in OK |
| --- | --- | --- |
| Avatar stack ("who's in the room") | light | **Yes** — extend the existing PresenceBar to humans |
| Live cursor + name label | medium | **Yes** — humans only (y-prosemirror decorations) |
| Selection highlight | medium | Yes — humans; agent equivalent is write-flash |
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

## Part 3 — Proposed collaboration data model

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

The spec frames it as in-CRDT (heavier, conflict-free) **vs** side store (lighter, anchoring harder). The survey says the answer every mature tool reached is **hybrid — split the thread from its anchor:**

- **Anchor → in the CRDT** as a `Y.RelativePosition` (or a ProseMirror mark via `y-prosemirror`). It rides concurrent edits conflict-free, survives agent full-file rewrites, and degrades to "orphaned/context-lost" (Google-Docs behavior) rather than pointing at the wrong text.
- **Thread payload (body, author, resolved, replies, mentions) → a side store**, keyed by thread id. Lighter, queryable, notifiable, and doesn't bloat the document Y.Doc.
- **Anchor granularity:** a character range is most precise; a **block-level anchor (Notion-style)** is more stable under heavy editing. Given OK's ProseMirror block model, a block-or-range anchor is a reasonable v1 — decide with a stability test (spec's test plan already calls for "comment stays attached under concurrent edits").

This keeps the document CRDT clean while getting conflict-stable anchoring — the best of both the spec's options.

### Ephemeral vs persisted — the rule

- **Ephemeral (never persist, allow to be wrong):** cursor, selection, viewport, focus, typing, active/idle, ad-hoc reactions/pings. Losing these on disconnect is *correct*.
- **Persisted (durable, merged or queryable):** document content, comment threads + anchors, notifications, doc metadata (owner/last-edited/activity), identity, roles.
- **The trap:** persisting presence (stale ghost cursors) or putting comment bodies inline in the text CRDT (bloat + fragile). Keep the boundary sharp.

### Per-user / per-doc / per-workspace

- **Per-user:** identity (name, avatar, color), role, notification inbox, and *their* ephemeral presence.
- **Per-doc:** content, comment threads, durable metadata (owner, last-edited, activity state), the set of currently-present participants.
- **Per-workspace:** membership, roles/groups, and (open question #4) where permission enforcement lives — federated store vs git-delegated.

## Part 4 — Recommended patterns for Open Knowledge

1. **Extend the PresenceBar to humans** with avatar stack + active/idle; add live cursors + selection via `y-prosemirror` once the shared server is reachable. Reuse, don't rebuild.
2. **Build comments as the flagship Phase 2 surface** — gutter threads, reply/resolve/`@`-mention, **anchor in-CRDT + payload side-store**. This is the biggest gap and the highest user value.
3. **Add a per-user notification inbox** for mentions/replies — the second gap; small but expected.
4. **Keep one identity model** across humans + agents (writer-ID taxonomy) so the Timeline and comments read coherently — satisfies open question #5.
5. **Make view-only a real product state** (editor renders read-only; all write surfaces refuse) — minimal viewer/editor split first, RBAC later. Note the Liveblocks nuance: a viewer can still *broadcast presence*, so you can see their cursor without granting edit.
6. **Give Layer-2 git conflicts a first-class resolve UI**, visually distinct from conflict-free live edits — resolves open question #6.
7. **Progressive disclosure throughout** — quiet by default, depth on demand, so presence + comments + diffs + timeline don't clutter the editor (the proposal's stated drawback).

## Part 5 — Open questions & tradeoffs

- **Anchor granularity: range vs block.** Range = precise but fragile; block = stable but coarse. Needs a concurrent-edit stability test before committing (feeds the spec test plan).
- **Comment store location.** Side store where? The federated backend (Miles' track), a local `.ok/` store, or git-tracked? Ties to open question #4 and the federated-sync dependency.
- **Permission enforcement point (open #4).** Federated store owns roles vs git-delegated with a thin UI veneer. G4 blocks on this.
- **Suggestion model scope.** Full Google-Docs suggestion mode vs a minimal propose/accept. Value vs build cost.
- **Invite & identity flow.** Link / account / GitHub identity — how a person joins a shared session. Depends on the federated backend's auth. Does view-only imply a **hosted read view** (a shareable link, no OK install), bridging from today's `share_link` (pull-into-your-own-install) model?
- **Notification transport.** In-app inbox only, or email/OS notifications too? Cadence/noise control overlaps open question #1.
- **Presence at scale.** Awareness broadcasts to all peers; large rooms need throttling/cursor-culling. Not urgent, but a known Yjs sharp edge.

## Links

- Hub: [[README|Collaboration UX]]
- North star: [[proposals/0001-collaboration-ux|Proposal 0001 — Collaboration UX]] (open questions #3–#6 are answered/advanced here)
- Phase 2 spec: [[specs/human-to-human-collaboration/spec|Human-to-human collaboration]] · [[specs/human-to-human-collaboration/tasks|tasks]]
- Phase 1 (the legibility model this reuses): [[specs/agent-change-visibility/spec|Agent change visibility]]
- Liveblocks docs — <https://liveblocks.io/docs> (Presence, Storage, Comments, Notifications, Permissions)
- Yjs shared types & awareness — <https://docs.yjs.dev>
- `y-prosemirror` (remote cursors + relative positions) — <https://github.com/yjs/y-prosemirror>
