---
title: Notion — block-anchored collaboration
description: Product study of Notion for Open Knowledge Phase 2 — block-level (vs character-offset) comment anchoring for stability under edits, lightweight avatar presence, and durable per-page collaboration metadata.
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
  - notion
  - comments
  - metadata
  - phase-2
---
# Notion — block-anchored collaboration

**Part of:** [[guides/collaboration-synthesis|Human-to-human collaboration research]] · **Hub:** [[README|Collaboration UX]]

Notion is surveyed for its **block-anchored** comment model and its durable per-page metadata.

- **Comments anchor to a block** (or an inline range within a block), not a character offset. A coarser anchor = more stable under edits, at the cost of precision. A pragmatic middle ground worth considering given OK's block-ish ProseMirror model.
- Presence = avatar chips + lightweight "editing" cursors; not the [[guides/figma|Figma]] spectacle.
- **Per-page metadata** (last-edited-by/at, owner, properties) is surfaced in the UI as durable collaboration state — exactly the "metadata related to collaborative docs" Nick asked about.

**For OK:** the block-vs-range anchor decision this feeds is resolved in the [[guides/collaboration-synthesis|synthesis data model]], and it lines up with [[guides/live-blocks|Liveblocks]]' anchor-in-CRDT/payload-side-store split.

## Links

- Research hub: [[guides/collaboration-synthesis|Human-to-human collaboration research]]
- Related: [[guides/live-blocks|Liveblocks case study]] · [[guides/figma|Figma]] · [[guides/google-docs|Google Docs]] · [[guides/yjs-baseline|Yjs/Hocuspocus baseline]]
