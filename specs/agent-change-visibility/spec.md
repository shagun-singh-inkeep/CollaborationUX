---
type: spec
title: Agent change visibility & legibility
description: Make agent-driven document edits something a user can notice, understand, navigate, and control — the Phase 1 immediate win.
status: draft
owner: shagun@inkeep.com
created: 2026-06-30
parent_proposal: proposals/0001-collaboration-ux
tags:
  - spec
  - collaboration
  - ux
  - agents
  - phase-1
---
# Spec — Agent change visibility & legibility (Phase 1)

**Status:** draft · **Owner:** shagun@inkeep.com · **Parent:** [[proposals/0001-collaboration-ux|Proposal 0001]] · **Hub:** [[README|Collaboration UX]] · **Next phase:** [[specs/human-to-human-collaboration/spec|Human-to-human]]

## Problem statement

When an AI agent edits a document through the MCP / HTTP write path, text changes. The user needs to answer: *Did something just change? What changed? Which agent did it? Why? Is it correct? How do I undo just that?* OK already answers several of these well — the per-document **Timeline tab** is a unified human+agent feed with summaries, expandable diffs, and per-entry restore (see Current state). The remaining gaps are mostly about **noticing** changes without opening that tab, **richer diff rendering**, **catch-up** for what changed since you last looked, and a **cross-document** rollup. Until those close, OK's headline capability — agents as collaborators — is harder to trust and demo than it should be.

This spec defines the end-to-end UX that makes an agent's work **legible**: notice → understand → navigate → control — completing what the Timeline starts.

### Why this first

- It is the product differentiator (see [[proposals/0001-collaboration-ux|the proposal]]).
- The substrate is furthest along.
- It needs **no federation** — agents already write locally, so value ships independent of Miles' backend track.

## Current state (shipped vs gap)

References point at `inkeep/open-knowledge` (a separate repo from this planning project).

**Shipped / in flight — reuse, don't rebuild:**

- **Write side-channels.** `Y.Map('agent-flash')` (per-write attribution) and `Y.Map('agent-effects')` (bounded \~50-entry activity ring). Types in `packages/core/src/types/awareness.ts`.
- **Presence.** `packages/app/src/presence/` distinguishes `'human'` vs agent participants; `PresenceBar` renders avatars; principal identity gives real git names (spec `2026-04-27-principal-identity-in-presence`, Approved). Multi-agent overlap handled (`2026-04-21-multi-agent-presence`, Scope Frozen). Foundational pattern: `2026-04-08-presence-awareness-ux` (Approved).
- **Timeline tab (the big one).** `TimelinePanel.tsx` is a per-document edit-history tab that already renders a **unified, reverse-chronological human+agent feed** — "WIP writes from agents, principals (humans), the file watcher, the service, plus upstream syncs." Per-writer icons (User for humans; Claude/Cursor/Codex/Copilot/Cline/Windsurf for agents; HardDrive for disk; GitBranch for upstream), per-entry change summaries, expand-a-row → inline `ActivityPanelDiffView` with a unified/split toggle, and per-entry Restore (→ `POST /api/rollback`). Fetches `GET /api/history`, polls every 10s. A folder-level rollup also exists (`FolderTimelineCard.tsx`). **This is most of the "Understand" pillar, already shipped.**
- **Agent Activity panel.** A *separate*, agent-scoped surface: click an agent avatar → per-agent / per-file / per-burst diffs and per-file undo, read from per-session `Y.UndoManager` (specs `2026-04-23-agent-activity-panel`, `-ui`). Complements the Timeline; it's a focused agent diff/undo tool, not the unified feed.
- **Change summaries.** Legible "what & why" per write, shown on Timeline rows (`2026-04-21-agent-write-summaries`, `2026-04-21-agent-change-notes`).
- **APIs.** `/api/agent-activity`, `/api/agent-burst-diff`, `/api/metrics/agent-presence`.
- **In-doc write-flash** primitive exists from the presence-awareness work.

**Gaps this spec targets** (the Timeline tab already covers the core "Understand" pillar; these are what's left):

- **Notice without opening the Timeline.** The feed is great once you're looking at it, but there's no ambient in-document signal that pulls you in — you have to think to open the tab.
- ==**No "changes since I last looked."** The Timeline is a full polled list with no session high-water mark / unread state — a returning user can't tell what's new at a glance.==
- **Rendered-block diffs.** Timeline diffs are line/text-based (`react-diff-view`); they don't render the changed blocks as they look (formatting, images) the way a Notion-style updates feed does.
- **Cross-document rollup.** Timeline is per-document (and per-folder); ==there's no workspace-wide "everything that changed recently" view.==
- **Notification cadence is unsolved** — no agreed model for announcing changes without noise, especially multi-agent / high-frequency.
- ==**In-document navigation to a change** (jump from a Timeline row to that section in the body) is weak.==
- **Empty / first-run / multi-agent disambiguation** states are undefined.

## Goals

- **G1 — Notice.** A user can tell, without watching, that an agent changed the doc, and roughly how much, within seconds of returning to it.
- **G2 — Understand.** For any agent change the user can see *what* (legible diff), *who* (attributed agent identity), and *why* (the change summary) without reading the whole document.
- **G3 — Navigate.** From any listed change, one action takes the user to that location in the document.
- **G4 — Control.** Undo *just* a specific agent's change to a file (last, or all) in one obvious action; review-after-the-fact, not gated approval.
- **G5 — Scale.** All of the above stay legible with multiple concurrent agents and across multiple touched files.
- **G6 — Calm.** Signal scales with attention, not edit volume — quiet by default, depth on demand.

## Non-goals

- Human-to-human presence, comments, and review → [[specs/human-to-human-collaboration/spec|Phase 2]].
- Gated/approval-before-apply as the default model (auto-apply stays; "propose mode" is at most an open question).
- View-only / permissions (Phase 2).
- Character-level agent cursors — agents batch-write section-level diffs, so typing-cursor presence is the wrong metaphor (established in `2026-04-08-presence-awareness-ux`).
- Hosted multiplayer / federation (Phase 2 dependency).

## Design

### Surfaces & how they map to the journey

| Stage | Surface | Reuses | Net-new UX |
| --- | --- | --- | --- |
| Notice | In-doc **write-flash** + agent marker in the gutter/region; **presence bar** avatar activity; **unread badge** on the Timeline tab | flash primitive, PresenceBar | ambient in-doc signal; catch-up/unread count ("3 new since you looked") |
| Understand | **Timeline tab**: unified human+agent feed, summaries, expandable diffs, restore (already ships) | Timeline tab, ActivityPanelDiffView, summaries | **rendered-block diffs** (not just line diffs); cross-document rollup view |
| Navigate | **Jump-to-change** from a Timeline row to the body position; in-doc change markers are clickable | Timeline rows | document-position deep-linking for a specific entry |
| Control | **Restore** (per Timeline entry) + agent panel's **undo last / all on file** | Timeline restore, per-file undo via UndoManager | optional explicit "reviewed"/ack state |

### Key interaction decisions

- **Review-after-the-fact, not approval.** Changes auto-apply (CRDT). The control surface is *undo / steer*, never a merge gate. No-action == accepted (an explicit "reviewed" state is an open question, not a default).
- **Calm by default.** Flash is transient; the Timeline tab is opt-in; the only persistent ambient signal is a small unread count. Multi-agent bursts coalesce into per-agent rollups rather than N separate alerts (G6).
- **Attribution is identity-first.** Every change is tied to a stable identity (color + name + icon), consistent across the flash, the presence bar, and the Timeline. Built on the writer-ID taxonomy (`agent-*`, `principal-*`, etc.) — the same taxonomy the Timeline already renders.
- **Richer diffs, not new diffs.** The Timeline already shows line/text diffs with a unified/split toggle. The net-new is **rendered-block** diffs (changed blocks shown as they look, incl. images/formatting) for scannability — the summary still answers "why" so the diff needn't be read in full.
- **Catch-up is the missing primitive.** A returning user sees "what changed since I last looked" — a session-scoped high-water mark / unread state layered onto the Timeline (and `agent-effects`), surfaced as a count plus a filtered "new" view.

### States to design explicitly

- Empty (no agent activity yet) · first-run (teach the model once) · single change · high-frequency burst · multiple concurrent agents · change to a doc not currently open · post-undo · agent error/partial write.

## Open questions

Inherits #1 (cadence), #2 (review semantics), #5 (unified identity) from [[proposals/0001-collaboration-ux|the proposal]]. Spec-specific:

- Does "notice" need an out-of-document channel (OS / desktop notification) or is in-app enough for the first cut?
- Is the activity panel the home for catch-up, or is there a lighter inline "changes" ribbon at the top of an edited doc?
- How long does an agent stay "present" after its last write before the avatar fades?
- Do we show in-progress agent writes (streaming) or only settled bursts?

## Migration

Additive and incremental — no data migration; everything layers onto the shipped Timeline. Sequence: (1) add an ambient in-doc notice signal + Timeline unread badge; (2) add the catch-up high-water mark / "new" view; (3) rendered-block diffs in Timeline rows; (4) jump-from-row to body position; (5) cross-document rollup; (6) settle notification cadence. See [[specs/agent-change-visibility/tasks|tasks]].

## Test plan

- **Multi-client integration** is mandatory for anything touching observers/presence (single-client tests miss remote-peer divergence — per OK's own testing rules). Cover: one agent / one doc; multi-agent / one doc; one agent / many docs; change-while-doc-closed.
- **Legibility usability pass:** can a user correctly answer "what did the agent change and was it right?" from the Timeline alone, timed, without reading the doc — and can they tell what's *new* since last visit.
- **Calm check:** an agent making N rapid edits produces a bounded, coalesced signal (not N alerts).
- **Undo correctness:** "undo last on file" and "undo all on file" map exactly to the agent's bursts and preserve human edits (origin/attribution intact).
- **Accessibility:** presence/diff color is not the only attribution channel; panel is keyboard-navigable (defer to the accessibility-checklist skill).
