---
title: Figma — presence as the product
description: Product study of Figma's real-time presence UX for Open Knowledge Phase 2 — multiplayer cursors, follow/observation mode, the avatar stack, and the principle that spatial presence is pure ephemeral awareness.
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
  - figma
  - presence
  - phase-2
---
# Figma — presence as the product

**Part of:** [[guides/collaboration-synthesis|Human-to-human collaboration research]] · **Hub:** [[README|Collaboration UX]]

Figma is surveyed for its **presence** UX — the gold standard for "who is here, where are they looking."

- **Buttery multiplayer cursors** with name labels and per-user color; the reference implementation for live spatial presence.
- **Follow mode / observation mode:** click an avatar to lock your viewport to theirs. Spectating is a first-class, named state — the spec's optional viewport-follow (G1) is this.
- **Avatar stack** in the top bar = the canonical "who's in the room" surface, with a grey/inactive vs active distinction. This is the [[README|PresenceBar]] pattern.
- Ephemeral spatial presence (cursor x/y, current selection, viewport) is **never persisted** — pure awareness.

**For OK:** the awareness substrate for this already exists in the [[guides/yjs-baseline|Yjs baseline]]; the cross-product presence patterns and where they land are collected in the [[guides/collaboration-synthesis|synthesis]]. Compare with [[guides/live-blocks|Liveblocks]]' finding that a read-only client can still broadcast presence.

## Links

- Research hub: [[guides/collaboration-synthesis|Human-to-human collaboration research]]
- Related: [[guides/live-blocks|Liveblocks case study]] · [[guides/google-docs|Google Docs]] · [[guides/notion|Notion]] · [[guides/yjs-baseline|Yjs/Hocuspocus baseline]]
