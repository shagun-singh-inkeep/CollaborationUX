---
title: "Liveblocks: collaboration data-model case study"
description: Deep dive on Liveblocks as the anchor case study for Open Knowledge Phase 2 collaboration — its five-layer state taxonomy (presence, broadcast, storage, comments/threads, notifications), the presence-vs-storage split, the anchor-in-CRDT/payload-side-store comment model, and how each maps onto OK.
type: research
category: research
status: draft
owner: shagun@inkeep.com
created: 2026-07-01
last_verified: 2026-07-01
tags:
  - collaboration
  - liveblocks
  - data-model
  - presence
  - comments
  - phase-2
---
# Liveblocks: collaboration data-model case study

**Status:** draft · **Owner:** shagun@inkeep.com · **Hub:** [[README|Collaboration UX]] · **Part of:** [[guides/collaboration-synthesis|Human-to-human collaboration research]] · **Feeds:** [[specs/human-to-human-collaboration/spec|Phase 2 spec]] · **Answers open questions** #3 (comment storage), #4 (permissions), #5 (unified identity) in [[proposals/0001-collaboration-ux|Proposal 0001]]

## Why Liveblocks is the anchor case study

Liveblocks is collaboration *infrastructure* — its entire API is a taxonomy of "what kind of shared state is this," which is exactly the question [[specs/human-to-human-collaboration/spec|Phase 2]] has to answer. Its product *is* the collaboration data model, so it forces the exact content-vs-collaboration-state split OK needs to make.

That's why this doc is the deep case study, while the other tools in the [[guides/collaboration-synthesis|research survey]] — [[guides/figma|Figma]], [[guides/google-docs|Google Docs]], [[guides/notion|Notion]], and the [[guides/yjs-baseline|Yjs/Hocuspocus baseline]] OK already runs — are read for their UX *surfaces* rather than their data model.

## The layer taxonomy

The unit of collaboration is a **Room**; participants join a room and get five distinct state layers, each with different persistence and conflict semantics:

| Liveblocks layer | Persistence | Conflict model | Examples |
| --- | --- | --- | --- |
| **Presence** | Ephemeral — in-memory, dropped on disconnect | Last-write-wins per client, no merge | cursor position, selection range, `isTyping`, current focus |
| **Broadcast events** | Ephemeral — fire-and-forget, not stored | None (one-shot) | emoji reactions, "pinged you", toasts |
| **Storage** | Persisted, server-authoritative | Conflict-free (their own `LiveObject`/`LiveList`/`LiveMap` CRDT, **or** a Yjs doc) | the document / canvas / app data |
| **Comments / Threads** | Persisted in a managed side store | Thread-level, queryable | comment body, author, `resolved`, replies, `@`-mentions, reactions, `metadata` (the anchor) |
| **Notifications** | Persisted per-user | — | inbox, mention alerts, thread-reply subscriptions |

## What matters for OK

- **The presence vs storage split is the load-bearing idea.** Presence is deliberately lossy and cheap; storage is durable and merged. Trying to make one thing do both is the classic mistake. OK already respects this: **Yjs Awareness = presence, Y.Doc = storage** (see the [[guides/yjs-baseline|Yjs baseline]]).
- **Comments are NOT in the document CRDT.** Liveblocks keeps thread bodies/metadata in a separate managed store and anchors them into the text via the editor binding (Lexical / Tiptap / BlockNote plugins) using the editor's relative positions. So the *anchor* rides the CRDT; the *payload* does not. This is the single most important finding for open question #3.
- **Thread `metadata` is an open key/value bag** — teams stash the anchor (quote text, block id, or x/y for canvas), `resolved`, priority, etc. Flexible metadata on the thread is how they avoid a rigid comment schema.
- **Identity is resolved, not stored.** Liveblocks holds only a stable `userId`; the app supplies `resolveUsers` / `resolveMentionSuggestions` to hydrate names/avatars. OK's analog already exists: the **principal identity** (real git name/email) and the [[proposals/0001-collaboration-ux|writer-ID taxonomy]] (`agent-*`, `principal-*`, `file-system`, `git-upstream`).
- **Permissions are per-room, coarse:** `room:write`, `room:read`, `room:presence:write`, granted via access/ID tokens with `usersAccesses` / `groupsAccesses`. Notable: **read-only clients can still broadcast presence** (`presence:write` without `write`) — you can *see* a viewer's cursor even though they can't edit. Directly relevant to OK's G4 (view-only).

## How this feeds the OK data model

The load-bearing conclusion is the **comment-anchor decision** (open question #3): every mature tool converged on *anchor in-CRDT, payload side-store*, and Liveblocks makes that split explicit in its API. The full four-quadrant OK data model this drives — ephemeral vs persisted, per-user vs per-doc vs per-workspace — lives in the [[guides/collaboration-synthesis|research synthesis]] alongside the cross-product UX patterns and recommendations.

**Net for OK:** the substrate already provides Liveblocks' presence + broadcast + storage layers (via [[guides/yjs-baseline|Yjs/Hocuspocus]]). Phase 2's net-new build is concentrated in **comments/threads and notifications**, plus making live human edits reachable.

## Links

- Research hub: [[guides/collaboration-synthesis|Human-to-human collaboration: UX & data-model research]]
- Other product studies: [[guides/figma|Figma]] · [[guides/google-docs|Google Docs]] · [[guides/notion|Notion]] · [[guides/yjs-baseline|Yjs/Hocuspocus baseline]]
- North star: [[proposals/0001-collaboration-ux|Proposal 0001 — Collaboration UX]]
- Phase 2 spec: [[specs/human-to-human-collaboration/spec|Human-to-human collaboration]] · [[specs/human-to-human-collaboration/tasks|tasks]]
- Liveblocks docs — <https://liveblocks.io/docs> (Presence, Storage, Comments, Notifications, Permissions)
