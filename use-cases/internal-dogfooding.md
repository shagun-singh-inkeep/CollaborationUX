---
type: use-cases
title: Internal dogfooding use cases — SPEC 1:1s & All Hands on OK
description: "Two recurring internal meetings run inside OpenKnowledge, written as persona-by-persona user stories that stress-test the collaboration UX at small (1:1) and large (All Hands) scale."
status: draft
owner: shagun@inkeep.com
created: 2026-07-10
tags:
  - collaboration
  - ux
  - use-cases
  - dogfooding
  - personas
---
# Internal dogfooding use cases

**Status:** draft · **Owner:** shagun@inkeep.com · **Hub:** [Collaboration UX](../README.md) · **Feeds:** [Proposal 0001](../proposals/0001-collaboration-ux.md), [Phase 1 spec](../specs/agent-change-visibility/spec.md), [Phase 2 spec](../specs/human-to-human-collaboration/spec.md)

Two recurring Inkeep meetings, run **inside** OpenKnowledge instead of over a call + screen-share, so the collaboration UX is validated by the team's own real work — the shared OK doc *is* the meeting surface. Each use case is written from several POVs, because the same meeting looks different to the author, a peer, and a lead, and each POV exercises a different part of the [three pillars](../proposals/0001-collaboration-ux.md) (presence · change legibility · control).

The two scale deliberately in opposite directions:

- **SPEC 1:1s** — small & synchronous (2–3 people + an agent). Intimate, high-trust, edit-heavy.
- **All Hands** — large & broadcast (many people + an agent + async viewers). Read-heavy, presence-noisy, catch-up-driven.

> **Personas.** "You" = Shagun (author/owner). "A peer" = another IC editing alongside. "Nick / Robert" = leads/reviewers. "Jonathan" = a distinct **view-only / async** persona (reads the recap rather than editing live). Role mappings for the named people are provisional — confirm before treating them as fixed.

## Use case 1 — SPEC 1:1s run on OK

**Scenario.** A weekly spec-review 1:1. The spec lives in OK; you prep it (often with an agent drafting alongside), then walk your reviewer through it live in the same document, capturing decisions inline as you go.

| # | As… | I want to… | so that… | Pillar | Exercises | Today |
| --- | --- | --- | --- | --- | --- | --- |
| 1a | **you (author)** | see the agent's edits to my spec since I last opened it, summarized | I can walk in knowing what changed without re-reading the whole doc | Change legibility | [Agent change visibility](../specs/agent-change-visibility/spec.md); Timeline catch-up | Partial — Timeline ships; "since I last looked" high-water mark is a gap |
| 1b | **you (author)** | know my reviewer has actually opened the doc and where they're reading | I can drive the conversation to the section they're on | Presence | Live human cursors / focus | Gap — needs federated sync |
| 1c | **a peer (co-IC)** | edit the same section live without clobbering each other | we can co-write a tricky paragraph in the meeting | Presence + Control | Conflict-free live merge; "who's editing what" | Merge ships; live co-presence gap |
| 1d | **a peer (co-IC)** | drop an inline comment / suggestion on a line | I can raise a point without derailing the live edit | Control | [Comments & suggestions](../specs/human-to-human-collaboration/spec.md) | Gap — comments are entirely absent |
| 1e | **Nick / Robert (lead)** | review and comment async before the 1:1, and @-mention me | the live time is spent on decisions, not read-through | Control | [@-mentions](../guides/collaborator-identity.md); comment threads | Gap |
| 1f | **Nick / Robert (lead)** | get notified when the spec I reviewed changes again | I don't have to poll the doc to stay current | Change legibility | [Notification inbox](../guides/collaborator-identity.md) | Gap — no cross-doc inbox |

## Use case 2 — All Hands run on OK

**Scenario.** A company-wide All Hands on one shared OK doc: a running agenda + live notes. A few people present, most watch, questions and reactions land inline, an agent captures notes in real time, and people who missed it read the recap afterward.

| # | As… | I want to… | so that… | Pillar | Exercises | Today |
| --- | --- | --- | --- | --- | --- | --- |
| 2a | **you (note-taker)** | have an agent capture notes live while I run the doc | the record is written without me typing through the meeting | Change legibility | Agent-as-collaborator; [live agent writes](../specs/agent-change-visibility/spec.md) | Agent write path ships; live legibility polish is Phase 1 |
| 2b | **you (note-taker)** | keep the doc legible with 15+ people present at once | presence signal doesn't drown the content | Presence | Presence bar at scale; progressive disclosure | Gap — untested at broadcast scale |
| 2c | **a peer (attendee)** | drop questions / reactions inline as topics come up | I'm heard without interrupting the presenter | Control | [Comments / reactions](../specs/human-to-human-collaboration/spec.md) | Gap |
| 2d | **Nick / Robert (presenting lead)** | present from the doc and see who's actually following along | I can gauge the room and pace accordingly | Presence | Viewport-follow; participant list | Gap — needs federated sync |
| 2e | **Jonathan (view-only / async)** | read the doc without being able to edit it | I can consume the All Hands without risk of changing the record | Control | [View-only access](../guides/view-only-access.md) | Gap — no role model; access == GitHub repo access |
| 2f | **Jonathan (view-only / async)** | catch up on "what I missed" after the meeting | I get the outcome without watching a recording | Change legibility | Timeline catch-up; cross-workspace rollup | Partial — per-doc Timeline ships; rollup + unread state are gaps |

## What these two use cases cover together

- **Both agent and human collaboration** — the agent note-taker/drafter (Phase 1) and the humans reviewing, commenting, and following along (Phase 2) appear in the same doc, in one [unified timeline](../specs/human-to-human-collaboration/spec.md).
- **Both scales** — the 1:1 pressure-tests intimate co-editing and review; the All Hands pressure-tests presence-at-scale, view-only, and async catch-up.
- **The biggest known gaps surface immediately** — comments/suggestions (absent today), view-only ([Jonathan](../guides/view-only-access.md)), "changes since I last looked," and the [cross-doc notification inbox](../guides/collaborator-identity.md) all show up as blockers the moment you try to actually run these meetings on OK.

Status of each referenced capability is tracked in [what's built & what's next](../guides/collaboration-status.md).

## Open questions

- Confirm the real roles for Nick, Robert, and Jonathan — is Jonathan genuinely view-only, or a different editing persona? The persona mapping above is provisional.
- Should these become **acceptance scenarios** for the Phase 1 / Phase 2 specs ("we can run our real SPEC 1:1 on OK") rather than just illustrative use cases?
- Are there other recurring internal meetings worth adding (standup, planning, retro) that would exercise a different corner of the UX?
