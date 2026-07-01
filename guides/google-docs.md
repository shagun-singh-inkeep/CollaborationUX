---
title: Google Docs — comments & suggestions
description: Product study of Google Docs for Open Knowledge Phase 2 — range-anchored comment threads in a gutter, anchoring that survives edits (graceful orphaning), suggestion mode with accept/reject, and lightweight caret presence.
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
  - phase-2
---
# Google Docs — comments & suggestions

**Part of:** [[guides/collaboration-synthesis|Human-to-human collaboration research]] · **Hub:** [[README|Collaboration UX]]

Google Docs is surveyed for its **comment-thread** and **suggestion** surfaces.

- **Comment threads anchored to a text range**, shown in a right margin/gutter; reply, resolve, `@`-mention (which emails/notifies). A resolved thread disappears from the doc but is retrievable — a persisted side store, not inline text.
- **Anchoring survives edits** via internal anchor/named-range IDs; when the anchored text is deleted the comment goes "orphaned"/context-lost rather than vanishing. A good model for OK's degrade-gracefully behavior.
- **Suggestion mode:** edits render as tracked insert/delete proposals with per-suggestion accept/reject. This is the heavyweight end of the suggestion open question.
- Presence is lighter than [[guides/figma|Figma]]: a colored **caret + name flag** and avatar chips, no follow-mode by default.

**For OK:** the range-vs-block anchor tradeoff and the side-store decision this informs are worked through in the [[guides/collaboration-synthesis|synthesis data model]]; the underlying anchor primitive (`Y.RelativePosition`) lives in the [[guides/yjs-baseline|Yjs baseline]].

## Links

- Research hub: [[guides/collaboration-synthesis|Human-to-human collaboration research]]
- Related: [[guides/live-blocks|Liveblocks case study]] · [[guides/figma|Figma]] · [[guides/notion|Notion]] · [[guides/yjs-baseline|Yjs/Hocuspocus baseline]]
