---
title: Notion — block-anchored collaboration
description: "Product study of Notion for Open Knowledge Phase 2 — its block-anchored comment UX and lightweight presence, plus its data model: a tree of block records with comments that reference a stable block UUID and durable per-page metadata."
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
  - data-model
  - phase-2
---
# Notion — block-anchored collaboration

**Part of:** [[guides/collaboration-synthesis|Human-to-human collaboration research]] · **Hub:** [[README|Collaboration UX]]

Notion is surveyed for its **block-anchored** comment model, its durable per-page metadata, and the **data model** underneath both — a tree of block records where comments reference a stable block UUID.

- **Comments anchor to a block** (or an inline range within a block), not a character offset. A coarser anchor = more stable under edits, at the cost of precision. A pragmatic middle ground worth considering given OK's block-ish ProseMirror model.
- Presence = avatar chips + lightweight "editing" cursors; not the [[guides/figma|Figma]] spectacle.
- **Per-page metadata** (last-edited-by/at, owner, properties) is surfaced in the UI as durable collaboration state — exactly the "metadata related to collaborative docs" Nick asked about.

## Data model

- **Everything is a block.** A Notion document is a tree of **block records**, each with a stable UUID, a `type`, a property bag, and an ordered list of child block IDs. Pages (and database rows) are themselves blocks. Storage is record/transaction-based and server-authoritative, not a character-level CRDT.
- **Comments reference a block ID.** A discussion/comment is a separate record that points at a **block UUID** (plus an optional inline range within the block). Because the anchor is a stable block ID rather than a character offset, it survives most edits — the coarse-but-stable anchor tradeoff.
- **Per-page collaboration metadata is durable record state** — last-edited-by/at, created-by, owner, and page properties are first-class fields on the block/page record, not ephemeral awareness.

Maps to OK: the stable block-ID anchor is the pragmatic middle between Google Docs' character-range anchor and a purely inline marker; it lines up with OK's block-ish ProseMirror nodes and the [[guides/live-blocks|Liveblocks]] anchor-in-CRDT/payload-side-store split.

**For OK:** the block-vs-range anchor decision this feeds is resolved in the [[guides/collaboration-synthesis|synthesis data model]], and it lines up with [[guides/live-blocks|Liveblocks]]' anchor-in-CRDT/payload-side-store split.

## Links

- Research hub: [[guides/collaboration-synthesis|Human-to-human collaboration research]]
- Related: [[guides/live-blocks|Liveblocks case study]] · [[guides/figma|Figma]] · [[guides/google-docs|Google Docs]] · [[guides/yjs-baseline|Yjs/Hocuspocus baseline]]
