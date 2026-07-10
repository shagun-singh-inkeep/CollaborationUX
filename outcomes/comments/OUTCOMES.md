# Outcomes: Commenting — anchored threads humans and agents share

**Last verified:** 2026-07-10
**Traces to:** [Make Open Knowledge collaboration legible and multiplayer](../human-collaboration-foundations/OUTCOMES.md) — carved out of that artifact's **O8** (anchored comments agents act on) + **O4** (human threads). This is the skill's "separable bet" split; the parent's O8/O4 become pointers to this artifact.
**Appetite:** open-ended (shaping, no time box) — inherited.

## Context

**SCR:**

- **Situation:** OK already gives agents the strongest change-legibility substrate on the market (live flash, dual-channel presence, per-agent activity, durable commit-grained attribution) but has **zero human comment primitives**. In the parent collaboration artifact, comments were shaped as two outcomes: O8 (`local` — anchored comments your agents read/reply/act on, buildable today over local MCP) and O4 (`hosted` — human-to-human threads on that same substrate). A 2026-07-10 design exploration ([comment-model-2026-07-10](comment-model-2026-07-10), synthesized from the [captures in this folder](ideas)) sharpened a concrete, differentiated model — a unified Comment/Ask-AI composer and comment-directed agent edits that land as one traceable, undoable change — detailed enough to spec on its own.
- **Complication:** the model now spans (a) a **local hero flow** buildable today — act on a selection to comment *or* direct an agent in place, and the agent answers by **editing the doc directly** — the change lands as one traceable, undoable edit in history and resolves the thread — and (b) a **hosted multiplayer half** (human delivery, mentions, comment-only roles) gated on the unshipped federated store. It settles the agent-edit-trust question the parent parked (PQ3: *observability, not gating*) on the side of O8: the comment-directed agent writes directly, and trust rides on a traceable, one-step-undoable edit rather than a pre-apply gate. Shipped un-sharpened, /spec would still re-derive the anchoring/storage boundary (the hardest engineering).
- **Resolution:** a dedicated, topology-tagged set of commenting outcomes — the local hero flow now (C1–C3), the hosted multiplayer half on federation (C4–C5) — inheriting the parent's locked decisions and the storage-fidelity invariant, with the agent-edit model decided here (CQ1: comment-directed edits land directly — observability, not gating).

**Grounding summary:** inherits the parent artifact's grounding wholesale — [current-state](../human-collaboration-foundations/evidence/current-state-collab-ux.md) (codebase reality: anchoring, attribution, timeline, gaps), [internal-specs-and-reports](../human-collaboration-foundations/evidence/internal-specs-and-reports.md) (locked decisions, house philosophy), [prior-art](../human-collaboration-foundations/evidence/prior-art-collaboration.md) (Notion/Figma/Docs/Obsidian/Linear/GitHub — table-stakes vs differentiators). New evidence here: [comment-model-2026-07-10](comment-model-2026-07-10) — the design-intent model from today's mockups. Load-bearing inherited locks: threads never pollute markdown or content-path sidecars (storage-fidelity contract); durable-artifacts-over-toasts; never guess identity, metadata-only attribution; one inbox with implicit subscription (parent PQ4); `/s/` reserved for cloud shares.

**What we're NOT doing (work-level):**
- [NOT NOW] Designing the anchor-storage / thread-persistence mechanism — that is /spec's job; this artifact carries only the invariant (no markdown pollution, anchors survive rebinds + merges).
- [NEVER] Building identity / membership / the member directory — consumed from the federated + member-management workstreams, never re-built here.
- [NOT NOW] Reactions/emoji, nested sub-reply trees beyond a flat resolvable thread, comment analytics — revisit after the core loop ships and is trusted.

## Items

| ID | Item | Type | Priority | Status | Notes |
|---|---|---|---|---|---|
| CQ1 | Agent-edit model: comment-directed edits land directly, no accept/discard gate | Product | P0 | Decided | **Decided (user-directed 2026-07-10): direct-act — observability, not gating.** An agent answering a comment makes the edit directly; it lands as one traceable, one-step-undoable edit and resolves the thread — no pre-apply Accept/Discard. Aligns C3 with parent O8 (agent writes directly) and PQ3 "observability not gating"; the trust surface is the traceable edit + undo/jump-to-change, not a gate. Owned by C3. Supersedes the accept/discard mockups in (comment-model-2026-07-10). |
| CQ2 | Unified composer: "same box, one switch" (Comment / Ask-AI, Team/AI toggle) | Product | P0 | Assumed | **Claim:** the two selection affordances collapse into one box; a Team/AI toggle picks recipient; pointed at AI, anchored text + note become the instruction (no separate prompt field). Confidence: **Medium** — clean design intent, feasibility unverified. **Verify by:** prototype the selection-composer against the editor selection API + the Ask-AI/MCP path in /spec. Owned by C2. (comment-model-2026-07-10) |
| CX1 | Storage fidelity: threads never in markdown or content-path sidecars; anchors survive CRDT rebinds + offline git merges, or orphan explicitly | Cross-cutting | P0 | Decided | **Inherited invariant (parent XQ3).** Storage *design* delegated to /spec of C1. Touches C1, C3, C4. |
| CX2 | Access tiers + GitHub boundary — "view / comment / edit, and how does that work with git?" | Cross-cutting | P0 | Parked | **Inherited (parent O6 + XQ4).** Commenting's slice: the commenter tier (C5) + threads carry no git representation (CX1), so comment-only never conflicts with git. Roles enforced at the hosted sync gateway, never in files. Trigger: hosted store + directory exist. (comment-model-2026-07-10) |
| CX3 | Identity + member directory dependency | Cross-cutting | P0 | Assumed | **Inherited (parent PQ1/XQ2/O2/TQ1).** Human authorship (C4), thread mentions, and roles (C5) all consume attested identity + an access-scoped member directory from the federated + member-management workstreams. Confidence: Medium. **Verify by:** confirm directory contract + owner before C4 → /spec. |
| CX4 | Inbox dependency for reply delivery | Cross-cutting | P0 | Assumed | **Inherited (parent O5/PQ4).** Human comment replies + thread mentions land in the one inbox; agent edits never notify. C4 ships alongside its inbox producer, not before. |

## Cross-cutting concerns

- **Storage fidelity (CX1)** — no markdown pollution, no content-path sidecars; anchors survive CRDT rebinds and offline git merges, or orphan explicitly. Touches C1, C3, C4. Owner: /spec, bounded here.
- **GitHub / access boundary (CX2)** — threads carry no git representation; roles enforced at the hosted sync gateway, never inside files; git-only collaborators see clean markdown and no comments (a deliberate trade-off to surface in sharing UX). Touches C4, C5.
- **Identity + directory (CX3)** — one access-scoped directory of attested workspace principals, reconciled with git-derived authors. Touches C4, C5. Owner: member-management + federated workstreams; this work states requirements only.
- **Inbox + notification philosophy (CX4)** — one inbox, implicit subscription, per-trigger opt-out; comment replies + thread mentions notify, agent edits never do. Touches C4.

## Outcomes — local track (buildable today)

No cloud, no other workstreams — agents connect over local MCP. This is the hero flow.

### C1 — A user can leave anchored comment threads on any doc `P0 · Now · [local]`
**Why:** Comments are where doc collaboration actually happens (table stakes: anchored threads, reply, resolve, defined orphan behavior — prior-art). For an agent-first KB the substrate is also the hardest and most reusable engineering: anchors that survive CRDT rebinds + offline git merges (CX1) are the same spike C3 and C4 both need — building it first, locally, **de-risks the entire comments space before federation exists** (platform). Standalone value today: a solo user annotates their own docs and leaves notes their agents can read (customer).
**Surfaces:** editor UI (in-doc anchor highlight, thread panel/popover, resolved state) — in scope; MCP (threads listable/readable by agents — the read half of C3) — in scope; share/export — [NOT NOW] threads stay workspace-internal, revisit with hosted shares; notifications — N/A locally (no recipient concept).
**Invariants:**
- Threads never appear in markdown or as content-path sidecars; the `.md` is byte-identical to a thread-free project and git push/pull is unchanged (CX1).
- An anchor survives CRDT rebinds and offline git merges, or **orphans explicitly** — never a silent loss.
- A resolved thread is retrievable, not destroyed; timeline restore/rollback never deletes threads.
**Non-goals:**
- [NOT NOW] Human-to-human delivery — that is C4 (needs identity + inbox).
- [NOT NOW] Thread notifications — no recipient concept locally; the thread is the durable artifact.
- [NEVER] Storing thread text inside the `.md` file or a per-doc sidecar.
**Acceptance criteria:**
- User can anchor a thread to a text range in any doc, reply, and resolve.
- Deleted anchor text produces an explicit orphan state ("original content deleted" precedent), never a silent drop.
- With threads present, the markdown files are byte-identical to a thread-free project and git behavior is unchanged.
- A thread survives a doc edit that doesn't touch its anchor, and re-anchors or orphans deterministically when the anchored text changes.
**Assumptions:** anchor fidelity across rebinds + merges is achievable via CRDT relative positions. Confidence: **Medium** — same spike family as the parent's re-anchoring assumption (parent O1); shared machinery, one investigation.
**Connections:** foundation for C2, C3, C4; inherits CX1 (parent XQ3); C4 becomes an audience-expansion on this substrate, not a rebuild.

### C2 — A user can comment or direct an agent from one gesture on a selection `P0 · Now · [local]`
**Why:** The differentiated gesture — "the two things you *do with* a selection sit together": leave a comment (human) or direct an agent (AI), **one box, one Team/AI switch**. Pointed at AI, the anchored text + your note become the instruction — no separate prompt field, so directing an agent in place is as light as commenting (comment-model-2026-07-10). No shipping prior art fuses human-comment and agent-direction into one surface (prior-art differentiator). Value: "comment on this" and "have my agent fix this" become the same muscle memory (customer); both reuse the C1 thread as their container (platform).
**Surfaces:** editor selection toolbar (Comment + Ask AI in the actions cluster, divider from formatting; collapses to a bubble icon when tight; ⌘⇧M) — in scope; the send step (Team/AI toggle; ⌘↵ to send) — in scope; MCP — N/A (this is the human entry surface; the agent side is C3).
**Invariants:**
- A comment is a comment regardless of recipient — the same thread object; one toggle decides the audience (CX1 storage applies to both).
- Pointed at AI, there is no separate prompt field — anchored text + note *is* the instruction.
- The composer never writes to the doc directly — it opens a thread; any doc edit comes from the agent via C3, landing as one traceable, undoable edit.
- All composer copy routes through Lingui (inherited app i18n rule).
**Non-goals:**
- [NOT NOW] Rich slash-commands / prompt templates inside the composer.
- [NOT NOW] Multi-range anchors from one composer.
**Acceptance criteria:**
- From a text selection the user can open one comment box and either post a human thread or send the anchored text + note to an agent, chosen by a Team/AI toggle in that same box.
- The AI path requires no separate prompt field.
- The affordance collapses gracefully to a bubble icon when toolbar space is tight; ⌘⇧M opens, ⌘↵ sends.
**Assumptions:** CQ2 (composer feasibility, Medium).
**Connections:** depends on C1 (needs a thread to post into); feeds C3 (the AI send path); the Team path feeds C4 under federation.

### C3 — An agent answers a comment by making the edit directly `P0 · Now · [local]`
**Why:** The on-brand payoff: you direct an agent on an exact passage and it **makes the change directly** — the edit lands as one edit in history (undo stepper / jump-to-change) and it replies *inside the thread* recording what it did, then resolves. This is the trust surface **and** the differentiator in one feature — every edit is traceable to the comment that caused it and reversible in one undo step (observability, not gating — parent PQ3), so directing an agent in place is as light as leaving a comment. No shipping prior art wires comment-directed agent edits into a markdown KB's history this way (prior-art differentiator).
**Surfaces:** thread UI (agent reply recording the edit, "asked AI" / "made an edit" attribution, link to the change in history) — in scope; MCP (agent lists/reads threads, posts a reply, makes the edit) — in scope; timeline/history (the edit lands as one attributed edit with jump-to-change + one-step undo) — in scope; notifications — N/A locally.
**Invariants:**
- A comment-directed agent edit lands **directly** — no Accept/Discard gate; trust comes from traceability + one-step undo, not a pre-apply review (observability, not gating — parent PQ3).
- The edit is **one edit** in history, attributed and traceable back to its thread (act↔thread traceability, inherited from parent O8), and reversible in one undo step.
- The agent's thread reply records the edit and links to the change; resolving the thread never destroys that link.
- Reply-before-act ordering (parent PQ5) — an agent posts its thread reply as it makes the acting edit, never a silent mutation.
**Non-goals:**
- [NOT NOW] Agent auto-resolving threads without making an edit or leaving a reply.
- [NOT UNLESS] One comment producing edits across multiple docs — revisit if multi-doc refactors via comments become a real need.
- [NOT NOW] A pre-apply Accept/Discard gate on comment-directed edits — CQ1 is decided for direct-act (observability, not gating); a review gate is out of scope here.
**Acceptance criteria:**
- An agent can list + read threads over MCP.
- From a comment directed at AI, the agent makes the edit directly and posts a thread reply recording the change and linking to it.
- The edit lands as a single edit visible in the timeline / undo stepper, resolves the thread, and is reversible in one undo step.
- The edit is traceable to its originating thread and feeds the parent's jump-to-change loop (parent O1).
- A multi-paragraph edit is presented coherently in the thread + history (loading + multi-block presentation is /spec's to design — flagged here, not designed).
**Assumptions:** CQ1 decided (direct-act); a comment-directed edit over the current agent-write / MCP path lands as one edit traceable back to its thread. Confidence: **Medium** — verify the thread↔edit wiring against the server-side agent-markdown-write + MCP write surface in /spec.
**Connections:** depends on C1 (thread) and C2 (the AI send path); owns CQ1; closes the loop with parent O1 (comment → agent edits directly → traceable change you can jump to and undo); under federation, the same direct-edit loop is visible to human collaborators (rides C4).

## Outcomes — hosted track (requires the federated central store)

### C4 — Collaborators discuss docs in shared anchored threads `P0 · Next · [hosted]`
**Why:** Comments become multiplayer — collaborators discuss a doc in the same anchored, resolvable threads with **attested** authorship, replies notify, and mentions pull the right person in. Table stakes for any shared-doc tool (prior-art). Here it is an **audience-expansion** of the already-working C1 substrate + C3 loop (agent edits stay visible and traceable to human collaborators too), not a rebuild.\
**Constraints:** depends on CX3 (attested identity + directory) and CX4 (inbox); threads never travel through git (CX2) — git-only collaborators see clean markdown, an explicit sharing-UX trade-off; anchors re-derive or orphan after a git-pull the comment store never saw as CRDT ops; timeline restore/rollback never deletes threads; threads follow docs across git renames/moves (doc-identity mapping, not path binding).\
**Connections:** depends on C1 substrate, CX3, CX4; carries mentions (parent O3); the C3 comment→edit loop extends to human collaborators.\
**Promote when:** federated backend + member directory reach spec/build — then fully sharpen (authorship, delivery, degraded/offline states) here.

## Sequencing

**Now — C1 + C2 + C3 (the local hero flow).** *dependency-first*: none depend on the unshipped federated backend, and C1's substrate is what C4 builds on; *risk-first*: C1 exercises the hardest assumption in the whole comments space (anchor survival across rebinds + git merges) before federation; *value-first*: together they close a loop — select → comment or Ask AI → agent edits directly → one traceable edit you can jump to and undo. **Walking-skeleton:** if federation slipped indefinitely, C1+C2+C3 still ship a differentiated, standalone comment-with-your-agents experience. **Barrel check:** three Now outcomes, but they form a dependency chain (C1→C2→C3) — effectively one sequenced stream, under capacity.

**Next — C4** with the first federated milestone (attested identity + inbox). Extends working threads to human recipients; the C3 comment→edit loop becomes visible to human collaborators.

**Later — C5.** Promote when the hosted store + directory exist and a non-editor persona is real.

**Cross-workstream gates:** C4/C5 are blocked by (a) the federated-collaboration-sync backend (Draft spec, Andrew) and (b) the member-management directory contract (CX3 — verify owner + contract before C4 → /spec).

## Rabbit holes

- **Designing comment storage/anchoring in this artifact or early /spec chatter.** Tempting — CX1 is intellectually the crux; it's /spec's job. Carry only the invariant (no markdown pollution, anchors survive merges).
- **Re-opening CQ1 into a full agent-governance redesign.** CQ1 is decided (direct-act, observability not gating) and C3 ships the direct-edit loop; don't re-litigate gating unless the parent PQ3 trigger (multi-human unreviewed edits) actually fires.
- **Notification preference matrices.** One inbox + implicit subscription + per-trigger opt-out, nothing more at v1 (parent PQ4).
- **Git-syncing threads for "offline collaborators."** Considered and rejected (async arrival, merge conflicts, untrusted identity, no notification); threads stay in the hosted store (CX2).
- **Rich comment features** (reactions, nested sub-reply trees) before the core loop is trusted.

## Pre-mortem

- **Most likely:** the anchor-fidelity spike (CX1) proves harder than the CRDT-relative-position assumption implies — orphaning is noisy, anchors drift after git merges. Mitigation: it's the first thing C1 builds and the risk-first reason it's Now; spike it before committing C2/C3 UX.
- **Second:** CQ1's direct-act decision proves too unguarded — a comment-directed agent makes a wrong edit that lands live with no pre-apply review. Mitigation: every such edit is one traceable, one-step-undoable edit tied to its thread and surfaced via jump-to-change (observability, not gating); revisit a review gate only if the parent PQ3 trigger (multi-human unreviewed edits) actually fires.
- **Third:** federation slips and "team comments" stalls at the local flow — expectation set as multiplayer, delivered as comment-with-your-agents. Mitigation: topology tags make the gate explicit; the local flow is real standalone value.

## Evidence & References

### Evidence Files
- [comment-model-2026-07-10](comment-model-2026-07-10) — design-intent model (unified composer; its accept/discard mockups superseded here by the direct-act decision — CQ1) + competitive captures
- [Untitled](ideas) — raw capture: screenshots (Notion, Mintlify, Coda, Claude, GitHub PR) + notes
- [../human-collaboration-foundations/evidence/current-state-collab-ux.md](../human-collaboration-foundations/evidence/current-state-collab-ux.md) — codebase reality (inherited)
- [../human-collaboration-foundations/evidence/internal-specs-and-reports.md](../human-collaboration-foundations/evidence/internal-specs-and-reports.md) — locked decisions + house philosophy (inherited)
- [../human-collaboration-foundations/evidence/prior-art-collaboration.md](../human-collaboration-foundations/evidence/prior-art-collaboration.md) — table stakes / differentiators / deliberately-avoid (inherited)

### Upstream Artifacts
- [Parent: Make OK collaboration legible and multiplayer](../human-collaboration-foundations/OUTCOMES.md) — the umbrella; O8/O4 carve into this artifact
- federated-collaboration-sync SPEC (Draft, Andrew) — hosted multi-tenant sync + `TENANT → PRINCIPAL` identity (lives in the monorepo: `public/open-knowledge/specs/2026-05-20-federated-collaboration-sync/`)

