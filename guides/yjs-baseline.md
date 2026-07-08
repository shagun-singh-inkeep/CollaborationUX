---
title: Yjs / Hocuspocus — the substrate OK already runs
description: Baseline study of the Yjs/Hocuspocus stack Open Knowledge is built on — the Awareness protocol as presence, y-prosemirror remote cursors, Y.RelativePosition for conflict-stable comment anchors, and the CC1 broadcast channel — showing Phase 2 is largely wiring, not inventing.
type: research
category: research
status: draft
owner: shagun@inkeep.com
created: 2026-07-01
last_verified: 2026-07-01
tags:
  - collaboration
  - research
  - yjs
  - hocuspocus
  - presence
  - data-model
  - phase-2
---
# Yjs / Hocuspocus — the substrate OK already runs

**Part of:** [[guides/collaboration-synthesis|Human-to-human collaboration research]] · **Hub:** [[README|Collaboration UX]]

OK is built on the same substrate the surveyed tools use under the hood, so much of Phase 2 is *wiring, not inventing* (echoing the spec's "shipped vs gap").

- **Awareness protocol = presence.** Ephemeral per-`clientID` state (cursor, selection, user identity/color), broadcast to peers, auto-cleared on disconnect. OK already carries a `'human'` vs agent discriminator here, and an `agentPresence` map for agents. This is the presence layer [[guides/live-blocks|Liveblocks]] keeps deliberately lossy.
- **`y-prosemirror`** ships remote-cursor and selection **decorations** out of the box — and OK **already has them wired and rendering**: `TiptapEditor.tsx` instantiates `yCursorPlugin` with a custom caret/label renderer (styled in `globals.css`), source mode draws remote carets via `y-codemirror.next`, and the `PresenceBar` surfaces human peers. Live human cursors (the [[guides/figma|Figma]] surface) are **already built and shipping**, not pending "turn-it-on" work.
- **`Y.RelativePosition`** — positions that survive concurrent edits. This is the primitive for conflict-stable comment anchors (the anchor half of the [[guides/collaboration-synthesis|comment-anchor decision]]), and what makes [[guides/google-docs|Google Docs]]-style graceful orphaning possible.
- **CC1 push-over-awareness** (OK's existing pure-signal broadcast channel) is the local analog of Liveblocks broadcast events — a place to hang ephemeral "X pinged you"/reaction signals without touching the doc CRDT.

**Implication:** OK's substrate already provides Liveblocks' presence + broadcast + storage layers. Phase 2's net-new build is concentrated in **comments/threads and notifications**, plus making live human edits reachable (the federated-sync dependency).

## Links

- Research hub: [[guides/collaboration-synthesis|Human-to-human collaboration research]]
- Related: [[guides/live-blocks|Liveblocks case study]] · [[guides/figma|Figma]] · [[guides/google-docs|Google Docs]] · [[guides/notion|Notion]]
