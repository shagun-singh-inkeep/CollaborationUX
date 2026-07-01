---
type: spec
title: Human-to-human collaboration
description: "Generalize the three collaboration pillars to people: live presence, comments/review, unified attribution, and view-only access — the Phase 2 spec."
status: draft
owner: shagun@inkeep.com
created: 2026-06-30
parent_proposal: proposals/0001-collaboration-ux
tags:
  - spec
  - collaboration
  - ux
  - human
  - phase-2
---
# Spec — Human-to-human collaboration (Phase 2)

**Status:** draft · **Owner:** shagun@inkeep.com · **Parent:** [[proposals/0001-collaboration-ux|Proposal 0001]] · **Hub:** [[README|Collaboration UX]] · **Builds on:** [[specs/agent-change-visibility/spec|Phase 1]]

## Problem statement

Two people cannot meaningfully collaborate on an Open Knowledge document in real time today. OK is local-first: the server binds to `localhost`, so the only co-editors reachable are other tabs on the same machine and AI agents. People who share a knowledge base collaborate **asynchronously through git** — they see each other's work only after a pull, never as it happens, and there is **no commenting, no review, and no notion of view-only access** anywhere in the product.

The machinery for live human collaboration is largely built — the CRDT merges conflict-free, awareness carries a `'human'` participant type, presence renders avatars — but it has no reachable shared server and no human-oriented collaboration surfaces on top. Phase 2 turns the latent capability into a product, reusing the legibility model proven in [[specs/agent-change-visibility/spec|Phase 1]].

## Current state (shipped vs gap)

**Shipped / in flight:**

- Conflict-free live merge (Yjs) — if two humans *were* connected to one doc, their edits already converge.
- Awareness + presence with a `'human'` discriminator and real principal identity (git name/email).
- **Unified human+agent Timeline already ships.** The per-document Timeline tab (`TimelinePanel.tsx`) renders principals (humans), agents, the file watcher, and upstream syncs in one reverse-chronological feed with summaries, expandable diffs, and restore. So the "unified timeline" pillar is **largely built** — humans already appear as first-class entries (today via async shadow-repo WIP commits; live human entries would flow the same way once federated sync lands).
- **Federated collaboration sync** — Inkeep-hosted central store (spec `2026-05-20-federated-collaboration-sync`, **Draft**, Miles' track). This is the reachable-shared-server dependency for *real-time* human presence/edits.
- Git-sync conflict subsystem (Layer 2): `sync-engine.ts`, `ConflictStore`, `/api/sync/conflicts`, `use-conflicts.ts` — conflicts exist and are tracked, but the resolution UX is thin.

**Gaps this spec targets:**

- **Live human presence in practice** — cursors, selection, and viewport-follow are not a usable feature on a default install (no reachable server).
- **Comments / annotations / suggestions** — entirely absent. The single largest collaboration-feature gap vs every comparable tool.
- **View-only / permissions** — no role model; access == GitHub repo access, not enforced inside OK.
- **Unified activity — mostly done; refine, don't build.** The Timeline already blends human + agent contributions per document. Remaining: making *live* human edits feed it coherently (not just async git commits), a cross-workspace rollup, and richer rendered-block diffs (shared with [[specs/agent-change-visibility/spec|Phase 1]]).
- **Conflict legibility** — git-sync conflicts surface weakly; no first-class resolve UX.

## Goals

- **G1 — Live human presence.** When two+ people edit the same doc over the shared backend, each sees the others' cursors, selections, and identity in real time; optional follow-a-person (viewport sync).
- **G2 — Comments & suggestions.** A person can attach a comment to a span or block, reply, resolve, and @-mention; optionally propose an edit as a suggestion that another can accept/reject.
- **G3 — Unified attribution (extend the shipped Timeline).** The Timeline already spans humans and agents; ensure *live* human edits feed it as first-class entries, add a cross-workspace rollup, and keep one identity model across the full writer-ID taxonomy. This goal is *refine*, not *build*.
- **G4 — View-only access.** A person can be given a document/project they can read but not edit, enforced by the product (not only by git permissions).
- **G5 — Conflict legibility.** When async git sync produces a conflict (Layer 2), the user gets a clear, resolvable UI — distinct from conflict-free live edits (Layer 1).

## Non-goals

- Re-deriving Phase 1's agent legibility model (this spec extends it).
- Building the federated transport itself (Miles' backend track — this spec consumes it).
- A full enterprise RBAC system; start with the minimal viewer/editor split (G4).
- Voice/video or chat-as-communication (Slack/Discord territory) — collaboration is *on the document*.

## Design

### Pillar mapping (reusing Phase 1)

| Pillar | Phase 1 (agents) | Phase 2 addition (humans) |
| --- | --- | --- |
| Presence | Agent avatars, focus, write-flash | **Live cursors + selection + follow-mode** — meaningful because humans type char-by-char (unlike batch-writing agents) |
| Change legibility | Timeline tab (unified human+agent feed) + per-agent diffs | **Live** human edits as first-class Timeline entries; cross-workspace rollup (timeline itself already exists) |
| Control | Undo last/all per file | **Comments, suggestions (accept/reject), view-only access** |

### Key interaction decisions

- **Cursors are for humans, diffs are for agents.** Humans get true real-time cursor/selection presence (Figma/Google-Docs metaphor); agents keep the burst-diff metaphor from Phase 1. The presence bar unifies both populations.
- **Comments anchored to content, conflict-free.** Comments attach to document ranges that survive concurrent edits — the anchoring-stability problem is the crux (see open questions). Threads support reply / resolve / @-mention.
- **Suggestions are optional, opt-in.** A lightweight "suggest instead of edit" mode produces an accept/reject affordance — the one place a gated model is appropriate, in contrast to Phase 1's auto-apply.
- **One timeline (already true).** The shipped Timeline tab is the shared feed; a human edit and an agent edit already read side by side. Phase 2 makes *live* human edits flow into it and adds a cross-workspace rollup — it doesn't introduce a new feed.
- **View-only is a real product state**, not just a repo permission — the editor renders read-only and write paths are refused for that participant. Enforcement point (federated store vs git-delegated) is an open question.

### States to design explicitly

Multiple humans + multiple agents at once · a viewer (read-only) participant · comment on text that later changes/moves · unresolved vs resolved threads · a git-sync conflict on pull · someone goes offline mid-edit · follow-mode when the followed person navigates away.

## Open questions

Inherits #3 (comment storage), #4 (permissions), #5 (unified identity), #6 (conflict UX) from [[proposals/0001-collaboration-ux|the proposal]]. Spec-specific:

- **Comment storage:** in-CRDT (anchored, conflict-free, heavier) vs side store (lighter, anchoring is harder). Drives anchoring stability.
- **Permission enforcement:** does the federated store own roles, or do we stay git-delegated with a thin UI veneer? G4 depends on the answer.
- **Suggestion model:** full suggestion mode (Google Docs) or a minimal propose/accept? Scope vs value.
- **Invite & identity:** how does a person get into a shared session — link, account, GitHub identity? Ties to the federated backend's auth.
- **Does view-only imply a hosted read view** (a shareable link someone opens without installing OK), bridging from the current `share_link` (pull-into-your-own-install) model?

## Migration

Gated on federated sync (`2026-05-20-federated-collaboration-sync`). Suggested order once the backend lands: (1) live human cursors/selection on the shared backend; (2) route live human edits into the existing Timeline + cross-workspace rollup (refine, not rebuild); (3) comments (the big rock); (4) view-only access; (5) first-class conflict-resolution UX. Comments and permissions can be specced in parallel with Phase 1 since they are largely independent of agent legibility. See [[specs/human-to-human-collaboration/tasks|tasks]].

## Test plan

- **Multi-client integration across machines/sessions** (not just multi-tab) for presence and live edit convergence.
- **Comment anchoring under concurrent edits:** a comment stays attached to its intended content when other participants edit around, inside, and across it; survives a full-file agent rewrite or moves predictably.
- **Permission enforcement:** a view-only participant cannot mutate via *any* write surface (editor, paste, drag, API).
- **Conflict UX:** induce a git-sync conflict (Layer 2) and confirm it surfaces distinctly from live edits and is resolvable in-product.
- **Unified-timeline coherence:** interleaved human + agent activity renders with correct, distinguishable attribution.
- **Accessibility:** cursors/labels and comment threads meet the accessibility-checklist bar; attribution never color-only.
