---
title: Google Docs — comments & suggestions
description: "Product study of Google Docs for Open Knowledge Phase 2 — its comment & suggestion UX (range-anchored threads in a gutter, graceful orphaning, suggestion accept/reject) and its data model: an OT document with comments and suggestions in an ID-anchored side store."
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
  - google-docs
  - comments
  - suggestions
  - data-model
  - phase-2
---
# Google Docs — comments & suggestions

**Part of:** [[guides/collaboration-synthesis|Human-to-human collaboration research]] · **Hub:** [[README|Collaboration UX]]

Google Docs is surveyed for its **comment-thread** and **suggestion** surfaces, and for its **data model** — an OT document with comments and suggestions kept in an ID-anchored side store.

- **Comment threads anchored to a text range**, shown in a right margin/gutter; reply, resolve, `@`-mention (which emails/notifies). A resolved thread disappears from the doc but is retrievable — a persisted side store, not inline text.
- **Anchoring survives edits** via internal anchor/named-range IDs; when the anchored text is deleted the comment goes "orphaned"/context-lost rather than vanishing. A good model for OK's degrade-gracefully behavior.
- **Suggestion mode:** edits render as tracked insert/delete proposals with per-suggestion accept/reject. This is the heavyweight end of the suggestion open question.
- Presence is lighter than [[guides/figma|Figma]]: a colored **caret + name flag** and avatar chips, no follow-mode by default.

## Data model

- **The document is an OT model, not a CRDT.** Google Docs uses **operational transformation**: each edit is an operation (insert/delete at an index) transformed against concurrent ops by a central server. Different lineage from OK's [[guides/yjs-baseline|Yjs]] CRDT, but the observable outcome — convergent concurrent editing — is the same.
- **Comments live in a side store, anchored by ID.** Thread bodies, authors, `resolved` state, and replies are not in the text stream; they attach via internal **anchor / named-range IDs** that the OT layer keeps updated as text shifts. Delete the anchored text and the thread **orphans** (context-lost) rather than vanishing.
- **Suggestions are overlay operations.** Suggestion-mode edits are stored as pending tracked insert/delete proposals with author + accept/reject state, layered *over* the base document rather than applied to it.

Maps to OK: the same anchor-in-document / payload-side-store split [[guides/live-blocks|Liveblocks]] makes explicit; the anchor primitive OK would use is `Y.RelativePosition` (see [[guides/yjs-baseline|Yjs baseline]]).

**For OK:** the range-vs-block anchor tradeoff and the side-store decision this informs are worked through in the [[guides/collaboration-synthesis|synthesis data model]]; the underlying anchor primitive (`Y.RelativePosition`) lives in the [[guides/yjs-baseline|Yjs baseline]].

## Links

- Research hub: [[guides/collaboration-synthesis|Human-to-human collaboration research]]
- Related: [[guides/live-blocks|Liveblocks case study]] · [[guides/figma|Figma]] · [[guides/notion|Notion]] · [[guides/yjs-baseline|Yjs/Hocuspocus baseline]]
