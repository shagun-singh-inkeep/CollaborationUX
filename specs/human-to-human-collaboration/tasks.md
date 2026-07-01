---
type: spec-tasks
title: Tasks — Human-to-human collaboration
description: UX task checklist for the Phase 2 human-to-human collaboration spec.
status: draft
created: 2026-06-30
tags:
  - spec
  - tasks
  - phase-2
---
# Tasks — Human-to-human collaboration

Checklist for [[specs/human-to-human-collaboration/spec|the Phase 2 spec]]. Hub: [[README|Collaboration UX]]. Gated on federated sync (Miles); comments & permissions can start in parallel with [[specs/agent-change-visibility/spec|Phase 1]].

## 0. Discovery & alignment

- [ ] Confirm federated-sync topology & timeline with Miles (`2026-05-20-federated-collaboration-sync`). *(blocking dependency)*
- [ ] Resolve proposal open questions #3 (comment storage), #4 (permissions), #6 (conflict UX).
- [ ] Decide minimal viewer/editor split vs richer roles for v1.

## 1. Live human presence

- [ ] Human cursors + selection on the shared backend (reuse awareness `'human'` type). *(backend touchpoint)*
- [ ] Identity labels on cursors via principal identity.
- [ ] Follow-a-person (viewport sync); handle followed-person-navigates-away.
- [ ] Presence bar unifies humans + agents.

## 2. Unified activity timeline (refine the shipped Timeline)

- [ ] Route *live* human edits into the existing Timeline tab as first-class entries (it already shows async human commits).
- [ ] Cross-workspace rollup view (Timeline is per-doc/per-folder today).
- [ ] Confirm one identity model across humans + agents (already rendered — don't regress).

## 3. Comments & suggestions (the big rock)

- [ ] Decide comment storage: in-CRDT vs side store. *(backend touchpoint — drives anchoring)*
- [ ] Range/block anchoring that survives concurrent edits + full-file agent rewrites.
- [ ] Thread UX: create, reply, resolve, @-mention.
- [ ] Optional suggestion mode (propose → accept/reject) — scope decision first.

## 4. View-only access

- [ ] Read-only editor render; refuse every write surface (editor, paste, drag, API). *(backend touchpoint: enforcement point)*
- [ ] Decide enforcement: federated store roles vs git-delegated.
- [ ] Explore a hosted read view / shareable link (bridge from current `share_link`).

## 5. Conflict legibility (Layer 2 / git sync)

- [ ] First-class resolve UX for `ConflictStore` conflicts (`use-conflicts.ts`). *(backend touchpoint)*
- [ ] Make git-sync conflicts visually distinct from conflict-free live edits.

## 6. Validation

- [ ] Cross-machine multi-client presence + convergence tests.
- [ ] Comment-anchoring tests under concurrent edits + agent rewrites.
- [ ] Permission enforcement test (view-only cannot mutate via any surface).
- [ ] Conflict-UX test (induce Layer 2 conflict; resolve in-product).
- [ ] Accessibility pass (cursors/labels/threads; attribution never color-only).
