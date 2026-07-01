---
type: proposal
title: Collaboration UX
description: "North-star framing for Open Knowledge's collaboration experience: agent change legibility first, human-to-human collaboration next."
status: draft
authors:
  - shagun@inkeep.com
created: 2026-06-30
tags:
  - proposal
  - collaboration
  - ux
---
# Proposal 0001 — Collaboration UX (north star)

**Status:** draft · **Author:** shagun@inkeep.com · **Created:** 2026-06-30

Parent hub: [[README|Collaboration UX]]. Graduates to: [[specs/agent-change-visibility/spec|Phase 1 spec]], [[specs/human-to-human-collaboration/spec|Phase 2 spec]].

## Motivation

Open Knowledge is a collaborative system that feels single-user. The CRDT (Yjs), the sync server (Hocuspocus), the awareness protocol, and a presence subsystem are all wired — but a person opening a document experiences a quiet text box. The product's actual differentiator, **agents editing alongside you**, is architecturally real and visually almost invisible.

Two forces make this the right thing to work on now:

1. **A unique, defensible wedge.** Every serious editor has multiplayer humans (Google Docs, Notion, Figma). Almost none has made an *AI agent a legible, first-class collaborator* in the document. That is OK's to win, and it is mostly a UX problem on top of substrate that already exists.
2. **A clean ownership split.** Miles owns the CRDT server backend (transport, sync, federation). Shagun owns the end-user collaboration UX. This proposal scopes the UX half so the two tracks can move in parallel against a shared model.

**The core insight:** *legibility precedes collaboration.* You cannot trust a multiplayer document — human or agent — if you cannot see what changed, understand who did it and why, and undo it. So we earn collaboration in two phases, hardest-and-most-differentiated first.

## Design

### The three pillars

Every surface in this workstream answers one of three questions. They are the spine of both phase specs.

- **Presence — "who/what is here?"** Avatars, cursors, focus, the live sense that you are not alone in the document.
- **Change legibility — "what changed, by whom, and why?"** Attribution, diffs, summaries, an activity timeline. The part OK is uniquely positioned to nail because agent writes carry structured intent.
- **Control — "can I review, undo, or steer it?"** Jump-to-change, undo last/all, accept/reject, follow-along, and (for humans) permissions.

### Phase 1 — Agent change visibility & legibility (the immediate win)

When an agent edits a document, the user should be able to **notice → understand → navigate → control** the change, whether it's one agent or several, one file or many. Detail: [[specs/agent-change-visibility/spec|the Phase 1 spec]].

This is deliberately first because (a) it's the differentiator, (b) the substrate is furthest along (agent-flash, activity panel, change summaries, presence), and (c) it sidesteps the hardest backend dependency — agents already write locally through the MCP/HTTP path, so no federation is required to ship value.

A load-bearing reframe: because CRDT edits **auto-apply**, agent-change UX is *review-after-the-fact* (like watching a teammate work), not gated approval. The control surface is undo/steer, not a merge gate. This is the opposite of a PR review and shapes every interaction.

### Phase 2 — Human-to-human collaboration (the generalization)

The same three pillars, extended to people: live human cursors and selection, comments and suggestions (OK has *none* today — the biggest product gap), and view-only vs edit access. The unified human+agent activity timeline mostly *already exists* (the Timeline tab) — Phase 2 refines it for live human edits and cross-workspace rollup rather than building it. Detail: [[specs/human-to-human-collaboration/spec|the Phase 2 spec]].

Phase 2 depends on the federated sync backend (Inkeep-hosted central store, Miles' track) because real-time human collaboration requires a reachable shared server — today OK binds to `localhost` and humans collaborate only asynchronously via git.

### Where OK sits vs the field

| Product | What they nailed | What OK borrows | Where OK differs |
| --- | --- | --- | --- |
| **Figma** | Multiplayer presence — buttery cursors, follow-mode | The presence bar; follow-an-agent | Participants can be agents, not just people |
| **Google Docs** | Comments, suggestion mode | Comment/suggestion model (Phase 2) | Local-first + git substrate, not hosted-only |
| **GitHub** | Async review on a git substrate | Change legibility, attribution, conflict surfacing | In-document and real-time, not PR-gated |
| **Cursor / Copilot** | AI diffs in an editor | Agent diff legibility | Agent is *in* the shared doc, not a side pane |

OK's position is the intersection nobody else occupies: **agent-native + local-first + git-backed + real-time.**

### Success criteria

- A user can answer "what did the agent just do to my doc, and was it right?" without reading the whole document.
- Undoing a specific agent change is one obvious action.
- Multiple agents (and later, humans) are individually distinguishable at a glance.
- The experience is legible without being noisy — change signal scales with attention, not with edit volume.

## Drawbacks

- **Surface-area creep.** Presence + diffs + summaries + an activity panel + notifications is a lot of UI; risk of a cluttered editor. Mitigation: progressive disclosure — quiet by default, depth on demand.
- **Phase 2 is backend-gated.** Real human multiplayer can't ship ahead of federated sync; we must avoid designing UX that assumes a topology the backend won't deliver.
- **Review-after-the-fact can feel like loss of control** to users who expect approval gates. We're betting that fast, legible undo beats slow approval — that bet should be validated early.

## Alternatives

- **Approval-gated agent edits** (stage changes; user accepts before they apply). Rejected as the default: it fights the CRDT's auto-apply model and the live-preview promise, and it's slow. May still appear as an opt-in "propose mode."
- **Side-panel agent diffs only** (Cursor-style), leaving the document untouched until accepted. Rejected as primary: it abandons the "agent is a collaborator *in* the doc" differentiator. The activity panel complements the in-doc experience; it doesn't replace it.
- **Lead with human multiplayer** (match Google Docs first). Rejected: it's the crowded, backend-heavy half and cedes the differentiator.

## Unresolved questions

These need Shagun↔Miles alignment; they gate the specs.

1. **Notification cadence & model.** Where do agent changes get announced (in-doc flash, panel badge, toast, OS notification) and how do we keep multi-agent / high-frequency edits from becoming noise?
2. **Review semantics.** Is there ever a gated/"propose" mode, or is it always review-after-the-fact with undo? Does "accept/keep" need to be an explicit state or is no-action == accepted?
3. **Comment storage (Phase 2).** Do comments live in the CRDT (anchored to positions, conflict-free, but heavier) or in a side store? This is a backend decision with large UX consequences — anchoring stability across edits is the crux.
4. ==**Permissions model (Phase 2).** View-only / edit — enforced where? Today access == GitHub repo access. Does the federated store introduce a real permission layer, or do we stay git-delegated?==
5. **Unified identity across humans + agents.** One attribution/timeline model spanning the writer-ID taxonomy (`agent-*`, `principal-*`, `file-system`, `git-upstream`) so the activity feed reads coherently.
6. **Offline / async reconciliation UX.** How do git-sync conflicts (Layer 2) surface and get resolved in the UI, distinct from conflict-free live edits (Layer 1)?

