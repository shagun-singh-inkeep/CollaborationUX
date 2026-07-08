---
title: Collaboration UX
description: Workstream hub for Open Knowledge's end-user collaboration experience — agent change legibility first, human-to-human collaboration next.
tags:
  - collaboration
  - ux
  - hub
---
# Collaboration UX

The end-user collaboration experience for [Open Knowledge](https://github.com/inkeep/open-knowledge) — making it feel like what it already is under the hood: a real-time, multi-participant, agent-native editor.

> **Ownership.** Miles leads the CRDT server backend (transport, sync, federation). Shagun owns the end-user collaboration **UX** — what a person actually sees, understands, and controls when someone (or something) else edits the document.

## The thesis in one line

Open Knowledge is *architecturally* collaborative (Yjs CRDT, Hocuspocus, awareness, presence — all wired) but *experientially* single-user. The work is closing that gap, starting with the half that's uniquely ours: **agents editing alongside you.**

## Scope — two phases

1. [[specs/agent-change-visibility/spec|Phase 1 — Agent change visibility & legibility]] (the immediate win). When an agent edits a doc, the user can *notice → understand → navigate → control* the change. Builds on substrate that already ships (presence bar, activity panel, write-flash, change summaries).
2. [[specs/human-to-human-collaboration/spec|Phase 2 — Human-to-human collaboration]] (the generalization). The same three pillars — presence, change legibility, control — extended to humans: live cursors, comments/review, shared attribution, view-only access. Rides on the federated sync backend.

Framing, principles, and roadmap live in the north-star RFC: [[proposals/0001-collaboration-ux|Proposal 0001 — Collaboration UX]].

## Three pillars (the spine of every spec here)

| Pillar | Question it answers | Phase 1 (agents) | Phase 2 (humans) |
| --- | --- | --- | --- |
| **Presence** | Who/what is here? | Agent avatars, focus, write-flash | Human cursors, selection, viewport-follow |
| **Change legibility** | What changed, by whom, and why? | Diffs + summaries + attribution in the activity panel | Unified human+agent timeline; comments |
| **Control** | Can I review, undo, or steer it? | Undo last/all per file; jump-to-change | Suggestions, accept/reject, view-only access |

## Current state (what already exists in the codebase)

Phase 1 is **not greenfield** — meaningful substrate has shipped or is in flight in `inkeep/open-knowledge`. These specs synthesize and complete it rather than reinvent it:

- `Y.Map('agent-flash')` + `Y.Map('agent-effects')` side-channels for agent write attribution and a bounded activity ring.
- Presence subsystem with a `'human'` vs agent discriminator (`packages/app/src/presence/`), `PresenceBar`, principal identity (real git names, not "Curious Squirrel").
- **Timeline tab** (`TimelinePanel.tsx`) — a per-document **unified human+agent feed** (summaries, expandable diffs, per-entry restore). Most of the "Understand" pillar already ships here.
- Agent Activity Panel (separate, agent-scoped: per-agent / per-file / per-burst diffs, per-file undo) + `DiffView`.
- Change summaries / change notes (legible "what & why" per write, shown on Timeline rows).
- APIs: `/api/agent-activity`, `/api/agent-burst-diff`, `/api/metrics/agent-presence`.
- Federated collaboration sync (Inkeep-hosted central store) — **draft**, the backbone for Phase 2.

A precise map of shipped-vs-gap lives in each spec's *Current state* section.

## Index

- [[guides/collaboration-status|Status — what's built & what's next]] — one-page, plain-language read of what already ships vs what's left, with how we'll build each remaining piece.
- [[proposals/0001-collaboration-ux|Proposal 0001 — Collaboration UX (north star)]] — vision, principles, competitive framing, roadmap.
- [[proposals/0002-contribute-from-ok|Proposal 0002 — Contribute from OK (fork-and-propose / in-product PR)]] — let a `pull`-only user send edits back to the source repo from inside OK; the missing "way back" GitHub-gated access needs.
- [[outcomes/human-collaboration-foundations/OUTCOMES|Outcomes — Human collaboration foundations]] — the Phase 2 foundations (comments, collaborator identity, view-only access) forced into sharp, testable outcomes with Now/Next/Later sequencing; sits between Proposal 0001 and the Phase 2 spec.
- [[specs/agent-change-visibility/spec|Spec — Agent change visibility & legibility]] · [[specs/agent-change-visibility/tasks|tasks]]
- [[specs/human-to-human-collaboration/spec|Spec — Human-to-human collaboration]] · [[specs/human-to-human-collaboration/tasks|tasks]]
- [[guides/collaboration-synthesis|Research — Human-to-human collaboration UX & data model]] — synthesis of the collaboration survey: presence + comment-thread patterns, a proposed ephemeral-vs-persisted data model (feeds Phase 2). Per-product studies: [[guides/live-blocks|Liveblocks]] · [[guides/figma|Figma]] · [[guides/google-docs|Google Docs]] · [[guides/notion|Notion]] · [[guides/yjs-baseline|Yjs/Hocuspocus baseline]].
- [[guides/view-only-access|Research — View-only access across tools]] — how Liveblocks, Google Docs, Notion, and Figma model read-only access, and what OK should adopt for Phase 2 G4.

_Workflow: proposals (RFC) graduate to specs (github/spec-kit triple), which graduate to `decisions/` once accepted._
