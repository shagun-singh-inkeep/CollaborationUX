---
type: spec-tasks
title: Tasks — Agent change visibility
description: UX task checklist for the Phase 1 agent change visibility & legibility spec.
status: draft
created: 2026-06-30
tags:
  - spec
  - tasks
  - phase-1
---
# Tasks — Agent change visibility & legibility

Checklist for [[specs/agent-change-visibility/spec|the Phase 1 spec]]. Hub: [[README|Collaboration UX]]. UX-owned; backend touchpoints flagged for Miles.

> **Baseline:** the per-document **Timeline tab** (`TimelinePanel.tsx`) already ships the unified human+agent feed — summaries, expandable line diffs, restore. These tasks *extend* it; they do not rebuild it.

## 0. Discovery & alignment

- [ ] Audit the shipped Timeline tab + agent activity panel + presence/flash end-to-end; screenshot the *current* notice→understand→navigate→control journey and mark what already works.
- [ ] Resolve proposal open questions #1 (cadence), #2 (review semantics), #5 (unified identity) with Miles.
- [ ] Decide where "notice" + catch-up live: ambient in-doc ribbon vs Timeline-tab unread badge vs both.

## 1. Notice (the missing pull)

- [ ] Ambient in-document signal that a change landed (gutter/region marker tied to write-flash) so users don't have to think to open the Timeline.
- [ ] Unread badge on the Timeline tab.
- [ ] Define agent-presence fade timing after last write.
- [ ] Settle notification cadence; multi-agent/high-frequency coalescing (per-agent rollup).

## 2. Catch-up ("changes since I last looked")

- [ ] Define the session high-water mark / unread state over the Timeline (+ `agent-effects`). *(backend touchpoint)*
- [ ] "New since last visit" filtered view on the Timeline.
- [ ] Behavior for changes to docs not currently open; cross-document rollup view.

## 3. Richer diffs (extend Timeline rendering)

- [ ] Rendered-block diffs (changed blocks shown as they look, incl. images/formatting) in addition to the existing line/split diff.
- [ ] Keep the change-summary line paired with each diff (already present — don't regress).

## 4. Navigate

- [ ] Jump from a Timeline row to the corresponding position in the document body. *(backend touchpoint: entry→position mapping)*
- [ ] Clickable in-doc change markers.

## 4b. Unify identity & attribution

- [ ] One identity object (name + color + icon) shared across flash, presence bar, and Timeline.
- [ ] Confirm the writer-ID taxonomy (`agent-*`, `principal-*`, `file-system`, `git-upstream`) maps cleanly to display identities (the Timeline already renders these). *(backend touchpoint)*

## 5. Control

- [ ] "Undo last on file" / "undo all on file" wired to per-session UndoManager; verify human edits preserved.
- [ ] Decide whether an explicit "reviewed/keep" state exists (vs no-action == accepted).

## 6. States & polish

- [ ] Design empty / first-run / single / burst / multi-agent / post-undo / agent-error states.
- [ ] Calm-by-default pass: signal scales with attention, not edit volume.
- [ ] Accessibility pass (keyboard nav; attribution never color-only).

## 7. Validation

- [ ] Multi-client integration tests (1 agent/1 doc; multi-agent/1 doc; 1 agent/many docs; change-while-closed).
- [ ] Timed legibility usability test ("what changed, was it right?" from the Timeline alone; can users spot what's *new*).
- [ ] Calm check (N rapid edits → bounded coalesced signal).
