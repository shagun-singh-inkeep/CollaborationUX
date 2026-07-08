---
title: "Outcomes: Make Open Knowledge collaboration legible and multiplayer"
description: "Finalized /outcomes artifact for the collaboration experience: six topology-tagged outcomes (agent change legibility now; attested identity, mentions, threads, inbox next; roles later) with items table, sequencing, rabbit holes, and pre-mortem."
tags:
  - outcomes
  - collaboration
  - phase-2
---

# Outcomes: Make Open Knowledge collaboration legible and multiplayer

**Last verified:** 2026-07-08
**Traces to:** User-initiated product direction (Shagun) — "collaboration experience of Open Knowledge: agent change indicators in the UI, human-to-human collaboration flows"
**Appetite:** (not yet specified)

## Context

**SCR (draft — pending user confirmation):**

- **Situation:** OK today is architecturally single-human-per-project: one local principal per project (git-config-seeded), no auth on the collaboration transport, and sharing that works as asynchronous *copies* over the git substrate. Agents, by contrast, are already first-class, *legible* editors: live line-flash on agent writes, a presence bar distinguishing agent "writing" from human "editing", a per-agent activity panel with burst diffs, and a timeline with durable per-commit attribution (5-way writer taxonomy, actor tuples, optional 80-char write summaries) (evidence/current-state-collab-ux.md). The federated-collaboration-sync spec (Draft, Andrew) proposes the hosted multi-tenant store and `app.inkeep.com`-backed identity but contains zero collaboration-UX decisions — personas and journeys are empty scaffolds (evidence/internal-specs-and-reports.md). Member invites and account/team management are owned by a separate workstream.
- **Complication:** Inkeep is building the cloud version, where multiple humans plus their agents share knowledge — but every human-to-human primitive is missing (no people directory, no comments, no notifications, no roles; @-mentions exist but mention *documents*, not people), and the missing pieces are gated by a trust boundary: awareness `principalId` and MCP `clientInfo` are client-published and loopback-trusted, with an explicit code note that non-loopback requires server-authoritative attribution. Meanwhile agent-change visibility — strong locally — lacks the "why" (no link from a write back to the conversation that produced it). Shipped as-is, cloud collaboration would be legible for agents and illegible for humans; and UX decisions made without pinning which topology each outcome targets (git-substrate today vs hosted store) will produce rework.
- **Resolution:** Shape the collaboration experience as a set of topology-tagged outcomes that (a) close the agent-legibility gaps on today's architecture, (b) consume identity from the federated principal model + member-management workstream rather than re-deciding it, (c) deliver the human primitives — mentions, comments/threads, notifications, roles — against the hosted store, and (d) fuse humans and agents into one presence/attribution fabric, which no shipping prior art offers (evidence/prior-art-collaboration.md).

**Grounding summary:** see evidence files — [current-state-collab-ux.md](evidence/current-state-collab-ux.md) (codebase reality), [internal-specs-and-reports.md](evidence/internal-specs-and-reports.md) (spec constraints + locked decisions), [prior-art-collaboration.md](evidence/prior-art-collaboration.md) (table stakes / differentiators / deliberately-avoid). Load-bearing locked decisions inherited: metadata-only upstream attribution, never guess identity; no sender attribution on share surfaces (D31); no cross-user identifiers in URLs (D21); `/s/` reserved for future cloud shares (D15); durable-artifacts-over-toasts house philosophy; attribution granularity floor = commit debounce.

**User's initial articulation (pre-contamination, captured verbatim in intent):**
- Work: the collaboration experience of Open Knowledge — (a) end-user experience of *agent* change indicators in the UI, (b) human-to-human collaboration flows.
- Pre-flagged problems to solve:
  1. **Collaborator identity** — who can you @-mention; leaning toward "Option 4 — Federated / OK-native accounts" (identity lives in OK's hosted backend; workspace membership = directory; the Notion/Figma model), which is already modeled in the Draft [federated-collaboration-sync spec](../../external-sources/federated-collaboration-sync-spec.md) as `TENANT → PRINCIPAL` with an `inkeep_identity` per principal.
  2. **Comments and threads.**
  3. **View-only** — open question whether it should exist at all ("would that be a thing??").
  4. **Some sort of notification model.**

**Strategic intake (Shagun, 2026-07-08):**

- **Why now:** Inkeep is starting to think about the *cloud version* of OK; better collaboration is part of that push. This work is UX-shaping for the cloud direction.
- **Sequencing vs. federation:** undecided. The federated-collaboration-sync backend has NOT shipped. Open question whether any of this lands on today's local-first/no-auth architecture or all of it assumes the hosted backend.
- **Appetite:** open-ended (shaping, no time box).
- **Adjacent work in flight:** member invites, account and team management are owned by someone else. This work must *consume* workspace membership/identity, not build or re-spec it.
- **Kill risk:** not yet articulated (pre-mortem to develop it).
  **Strategic intake (Shagun, 2026-07-08):**
- **Why now:** Inkeep is starting to think about the *cloud version* of OK; better collaboration is part of that push. This work is UX-shaping for the cloud direction.
- **Sequencing vs. federation:** undecided. The federated-collaboration-sync backend has NOT shipped. Open question whether any of this lands on today's local-first/no-auth architecture or all of it assumes the hosted backend.
- **Appetite:** open-ended (shaping, no time box).
- **Adjacent work in flight:** member invites, account and team management are owned by someone else. This work must *consume* workspace membership/identity, not build or re-spec it.
- **Kill risk:** not yet articulated (pre-mortem to develop it).**Strategic intake (Shagun, 2026-07-08):**
- **Why now:** Inkeep is starting to think about the *cloud version* of OK; better collaboration is part of that push. This work is UX-shaping for the cloud direction.
- **Sequencing vs. federation:** undecided. The federated-collaboration-sync backend has NOT shipped. Open question whether any of this lands on today's local-first/no-auth architecture or all of it assumes the hosted backend.
- **Appetite:** open-ended (shaping, no time box).
- **Adjacent work in flight:** member invites, account and team management are owned by someone else. This work must *consume* workspace membership/identity, not build or re-spec it.
- **Kill risk:** not yet articulated (pre-mortem to develop it).

## Items

| ID | Item | Type | Priority | Status | Notes |
|---|---|---|---|---|---|
| XQ1 | Topology tagging: which outcomes land on today's local-first/git architecture vs the federated hosted store? | Cross-cutting | P0 | Decided | **Directed (2026-07-08):** every outcome carries a `[local]` / `[hosted]` / `[both]` tag. Agent-legibility work is buildable today; all human-to-human primitives presuppose the hosted store (no recipient concept pre-federation). (evidence/current-state-collab-ux.md) |
| XQ2 | Interface with the member-invites / account & team management workstream | Cross-cutting | P0 | Assumed | **Claim:** that workstream delivers a queryable, access-scoped member directory (names, avatars, principal IDs) backed by app.inkeep.com identities, consumable by mentions/comments/notifications. Confidence: **Medium** (auth reuse is locked in the federated spec; the directory *contract* is unconfirmed — user doesn't know the owner/status). **Verify by:** identify the workstream owner and confirm the directory contract before /spec of O2/O3. Contract-shape hint from collab-infra prior art (Liveblocks): platforms take identity as a host-supplied *resolver interface* — resolve members by id + suggest mentionable members, with names/avatars, access-scoped — rather than owned data; that two-function interface is essentially what to ask the workstream for. |
| PQ1 | Collaborator identity model — Option 4 (federated / OK-native accounts) | Product | P0 | Assumed | **Claim:** identity will be OK-native/federated per the federated spec's TENANT → PRINCIPAL / inkeep_identity direction (user's lean; spec is Draft — model proposed, not confirmed). Treated as inherited direction + dependency, NOT this artifact's decision. Confidence: **Medium**. **Verify by:** confirm with federated-spec owner (Andrew) that the principal model survives spec finalization; revisit O2/O3 requirements if it changes. |
| PQ2 | Should view-only access exist? | Product | P0 | Parked | **User call (2026-07-08): keep open.** Reframe recorded: "view-only" = (a) share-a-copy (exists), (b) live view of a local server (needs auth), (c) hosted-store role. Lean: scope as hosted-store role with a *commenter* middle tier from day one (prior art: pain concentrates at viewer/commenter boundary). **Trigger to revisit:** hosted store + member directory exist, or O6 gets promoted. (evidence/prior-art-collaboration.md) |
| TQ1 | Server-authoritative actor attribution on the transport | Technical | P0 | Decided | **Directed (2026-07-08):** resolved by promotion to outcome O2 — every networked feature depends on it; no h2h outcome ships on client-published identity. (evidence/current-state-collab-ux.md) |
| TQ2 | Identity reconciliation: git-derived identity ↔ inkeep principal | Technical | P0 | Decided | **Delegated (2026-07-08):** the requirement is locked (one human = one rendered identity across timeline + directory; inherit "never guess, metadata-only"); the reconciliation design belongs to /spec of O2/O3. |
| PQ5 | How far do agents go in comment threads: read / reply / act? | Product | P0 | Decided | **Directed (2026-07-08, Shagun): "(c) or (b)"** — agent participation in scope up to acting on comments; ship order reply → act, act gated on change↔thread traceability (owned by O8 since the local/hosted split; O4 inherits it). |
| PQ7 | Timeline row semantics: authored delta vs restore preview | Product | P0 | Decided | **Directed (2026-07-08, user-surfaced via the "hi"→"hey" example):** a timeline row must show what THAT actor changed (`commit vs parent`, native in git), not today's `commit vs live` diff — which conflates every later editor's edits into one actor's row (the agent's "hi" appears as a removal once someone types "hey"). This is arguably a fix worth making independent of jump-to-edit. Restore-preview (`commit vs live`) remains as a separate explicit action next to Restore. **Feasibility (code-traced 2026-07-08): Moderate, two-tier.** Commit graph is a clean per-writer parent chain (`commit-tree -p parentSha`, `shadow-repo.ts:562-588`); park/checkpoint commits already excluded from the timeline. TIER 1 (small, ships the common case incl. the "hi"→"hey" example — sequential separate commits): switch the diff from `commit vs live` to `parent vs commit` via a `git show <sha>^:<path>` sibling of the existing `/api/history/:sha` handler + expose parent sha in TimelineEntry (`timeline.ts:9-26` carries only `sha` today). TIER 2 (harder, true per-actor isolation): the L2 drain commits ONE whole-content-dir tree shared across all writers in a \~15s window (`persistence.ts:843-906`), so parent-vs-commit still conflates any co-window writer; durable attribution stores doc names + summaries only, no byte ranges; the finer per-actor burst diffs exist but are ephemeral in-memory (`agent-activity.ts`, `agent-sessions.ts:516`), live sessions only. True isolation needs persisting burst diffs OR per-writer-scoped WIP trees — a write-path change. (evidence/current-state-collab-ux.md, authored-delta addendum) |
| PQ6 | Does O1 need a "why" link from a change to its originating conversation/task? | Product | P0 | Decided | **Directed (2026-07-08, Shagun): no** — cut from O1 scope; existing write summaries suffice as the "why". Demoted to [NOT NOW] non-goal on O1 with revisit triggers (summaries prove insufficient, or O4 act-on-comment lands). Side effect: kills O1's riskiest assumption (MCP conversation-reference capture). |
| PQ3 | Agent-edit trust: observability vs pending-review gating | Product | P0 | Decided | **Directed (2026-07-08): Option A — observability, no gating.** Double down on flash + presence + timeline + per-agent undo. Pending-review parked as **[NOT UNLESS]** — condition: multi-human workspaces where one member's agent edits other people's docs unreviewed (the calculus changes there; prior art says adding review later is safe, removing it isn't). User also added a sharpening requirement → O1: timeline entries must *jump to the edited part in the doc*, not only show a diff. |
| PQ4 | Notification philosophy: what notifies? | Product | P0 | Decided | **Directed (2026-07-08, no objection to stated intention):** mentions / comment replies / shares notify into ONE inbox with implicit subscription (Linear model); agent edits never notify by default — the timeline is their durable artifact (house philosophy). |
| XQ4 | Does GitHub stay canonical storage once the central server exists, or demote to an optional mirror? | Cross-cutting | P0 | Open | **User-raised (2026-07-08).** Git plays 3 roles: (a) real-time sync — NOT git even today (CRDT-over-WS owns this; server just extends its reach); (b) attribution/history substrate — STAYS (local git shadow drives O1's timeline); (c) storage + team-sharing backend — THE OPEN ONE. Federated spec leaves central persistence "model TBD" (Q5 central durability) — either git-backed central store (GitHub stays source of truth) or DB-backed (GitHub → optional export). Locked: local-first survives; the wire is CRDT ⇄ central, not git⇄git. Blocks nothing local (O1/O8 use the local shadow regardless); bounds how O4/O5 thread+inbox state persists centrally and whether O9's "refused/reconciled" reconciles against git or a DB. Owner: federated spec (Andrew) / Q5. (evidence/internal-specs-and-reports.md) |
| XQ3 | Comment storage must not pollute the markdown files | Cross-cutting | P0 | Decided | **Locked as invariant (2026-07-08):** thread/notification metadata never appears in user markdown or as sidecars in content paths (storage-fidelity contract); anchors must survive CRDT rebinds and offline git merges. The storage *design* is Delegated to /spec of O4/O5. |

## Cross-cutting concerns

- **Topology tags (XQ1)** — every outcome is tagged `[local]` / `[hosted]` / `[both]`; hosted-tagged outcomes are blocked by the federated-collaboration-sync backend shipping. Touches all outcomes. Owner: this artifact.
- **Member directory & identity (XQ2, PQ1, TQ2)** — mentions, comments, notifications, and roles all consume one access-scoped directory of workspace principals (inkeep-identity-backed), reconciled with git-derived authors in history. Touches O2–O6. Owner: member-management workstream + federated spec (Andrew); this work states requirements only.
- **Trusted attribution on the transport (TQ1)** — client-published identity is loopback-trusted today; every networked feature requires server-attested actors first. Touches O2–O6 (and O1 under federation). Owner: federated spec; surfaced here as O2.
- **Storage fidelity (XQ3)** — no sidecars in user-content paths; markdown files must stay byte-faithful; collaboration metadata (threads, notification state) needs a managed store whose anchors survive CRDT rebinds and offline git merges. Touches O8, O4, O5. Owner: /spec, bounded here.
- **Notification philosophy (PQ4)** — one inbox; mentions/comments/shares notify with implicit subscription; agent edits never notify by default (timeline is their durable artifact). Touches O3, O4, O5, O6.

## Outcomes — local track (achievable today)

Buildable on today's local-first architecture — no cloud, no other workstreams, start anytime. The hosted track below is gated on the federated central store (and the member-management directory) shipping.

### O1 — A human working alongside agents can trace any agent change to *what and who* — and jump to it in the doc   `P0 · Now · [both]`

<img src="/outcomes/human-collaboration-foundations/pasted-20260708-145511.png" />

<img src="/outcomes/human-collaboration-foundations/pasted-20260708-162034.png" />

**Why:** Trust in unattended agent edits is the adoption blocker for an agent-first knowledge base — and OK already owns the strongest substrate in the market (live flash, dual-channel presence, per-agent activity bursts, durable commit-grained attribution with actor tuples and write summaries; evidence/current-state-collab-ux.md). What's missing is *in-context navigation*: timeline entries show diffs but don't take you to the edited part of the document. Closing that gap converts existing observability plumbing into the differentiator no shipping product has — unified human+agent change legibility (customer trust) AND makes burst-level provenance durable, which is the same substrate a future review-gate or agent-thread-participation would need (platform) AND gives the cloud version its "watch your agents work" demo story (GTM). The "why" of a change stays at the existing write-summary level (PQ6 — user-directed): summaries already ship and render in the timeline; no new why-machinery in this outcome.
**Surfaces:** editor UI (timeline, activity panel, in-doc highlights) — in scope; MCP write path (provenance capture from agent clients) — in scope; share/export surfaces — [NOT NOW] provenance stays workspace-internal, revisit with hosted shares; notifications — N/A (agent edits don't notify, PQ4).
**Invariants:**
- Every agent write renders with an attributed actor (agent brand, session, principal) with zero user configuration — holds today, must survive federation. Falsifiable: no change row in a shared workspace ever shows "unknown actor".

- Provenance never gates or delays the write path (observability model, PQ3 — Directed).

- Durable attribution granularity floor = commit debounce (locked, timing report); burst-level provenance layered where available, never per-character as primary UI.

- Jump-to-region never dead-clicks: it lands on the live region when content exists, and explicitly says the content was deleted/moved when it doesn't.
**Non-goals:**

- [NOT UNLESS] Pending-review/accept-reject gating of agent writes — condition: multi-human workspaces where one member's agent edits others' docs unreviewed (PQ3).

- [NOT NOW] Per-character authorship coloring as primary attribution — prior art warns against; revisit only if commit/burst granularity demonstrably fails users.

- [NOT NOW] Cross-project/global agent identity — tracked in agent-identity worldmodel report.

- [NOT NOW] Deep link from a change back to the originating conversation/task (PQ6 — user-directed cut; also killed this outcome's riskiest assumption). Revisit if: write summaries prove insufficient to answer "why did the agent do this" in real usage, or when O8's act-on-comment lands (where the thread itself is the why).
**Acceptance criteria:**

- A timeline row shows what THAT actor changed (the authored delta, `commit vs parent`), not how the old version differs from now (today's `commit vs live` restore-preview conflates every later editor's changes into one actor's row — e.g. an agent's "hi" shows as a removal once someone edits to "hey"). Restore-preview stays available as a distinct "preview restore" action.
- From any timeline or activity-panel entry, one action scrolls the current doc to the corresponding region with an in-context highlight; deleted regions get an explicit "content no longer present" affordance (Docs orphaned-comment precedent) rather than a silent no-op.

- User can step prev/next through the changes of an agent session in-document (Docs "see new changes" / Word track-changes navigation pattern), not only read a diff pane.

- Every agent change durably exposes what (diff), who (agent + session + principal), and when — with the existing agent-supplied write summary rendered when present (no new "why" machinery).

- A user returning after hours away can reconstruct "what did agents do while I was gone" from durable UI alone — no reliance on toasts or having watched live. This is fully `[local]`: the git shadow repo persists every agent write + attribution to disk, read back via the local server's `/api/history` — no hosted dependency. Note the two-diff split: per-row = authored delta (PQ7); cumulative "since I left" = `stored-point vs live` (the restore-preview diff, which has a legitimate home HERE even though it's wrong for per-actor rows). Requires one small UX affordance — a "last seen" marker (or a checkpoint) to bound "while I was gone" without manual effort; the data + diff already exist locally.

**Done:&#x20;**

**Git commit attribution in timeline**

<img src="/outcomes/human-collaboration-foundations/pasted-20260708-161259.png" />

**Assumptions:**

- Jump-to-edit splits into two claims (code-verified 2026-07-08, recalibrated after the two-diffs distinction surfaced). Claim 1 — given current-doc positions, scroll + highlight is EASY: primitives exist (`EditorView.scrollIntoView`, `SourceEditor.tsx:86`; agent-flash `Decoration.line`). Confidence: **High**. Claim 2 — deriving the RIGHT positions is the actual work. Confidence: **Medium**. The timeline diff today is `diff(commit_snapshot, live)` (`use-timeline-entry-diff.ts:74`) = a *restore preview*, NOT what an actor changed: for "agent adds hi, then user changes to hey", expanding the agent's row shows `-hi +hey` (the agent's own content on the minus side, the user's later edit as the addition). Faithful actor attribution needs the *authored delta* (`commit vs parent`, native in git), whose positions are at commit-time and must be re-anchored to the live doc for jump-to-current — the real spike. Two bounded sub-unknowns: (a) WYSIWYG/TipTap needs a source-line → rendered-node mapping (the XmlFragment↔Y.Text bridge has the data); (b) deleted regions have no current line — detectable, and that detection IS the "content no longer present" affordance. Note: Notion proves the end-to-end UX at scale via stable block-UUID anchors (`#block-uuid` deep links from its Updates feed — see the addendum in [prior-art-collaboration](evidence/prior-art-collaboration.md)); OK's engineering delta is deriving equivalent durable anchors without a block model — CRDT relative positions are the natural analog.
**Connections:** traces to cloud-version push; extends `specs/2026-07-06-timeline-upstream-author-attribution` and the existing attribution fabric (precedent #25); O2 reuses the actor schema for humans; O4 (agents in threads) and the parked review-gate would build on the same durable provenance.

### O8 — A user can leave anchored comments on their docs that their agents read, reply to, and act on   `P0 · Now · [local]`

<img src="/outcomes/human-collaboration-foundations/pasted-20260708-154205.png" />

<img src="/outcomes/human-collaboration-foundations/pasted-20260708-171948.png" />

**Why:** Carved out of O4 (user-directed, 2026-07-08): the thread substrate and the agent loop need no cloud — agents connect over local MCP, so "threads for me and my agents" is fully buildable on today's architecture. It turns comments into a way to direct agents *in place* ("tighten this section, keep the citations" anchored to the exact paragraph) — the most on-brand version of comments for an agent-first product, with no shipping prior art ([prior-art-collaboration](evidence/prior-art-collaboration.md) differentiator #5). It also de-risks the hardest engineering in the whole comments space — XQ3 anchor survival across CRDT rebinds and offline git merges — before federation exists; when the hosted store lands, the same threads gain human recipients and O4 becomes an audience expansion, not a rebuild.
**Invariants:** XQ3 (threads never in markdown or content-path sidecars; markdown stays byte-faithful with threads present); anchors survive CRDT rebinds and offline git merges, or orphan explicitly; act-on-comment traceability — an agent edit made from a comment is traceable back to that thread (moved here from O4 with the split); agent participation follows PQ5 ordering: read → reply → act.

1. Where threads live: in the CRDT, beside the text — never in it
   The key move: a document's CRDT (its Y.Doc) can hold more than one thing. Today it holds the source text (which the persistence layer writes to the .md file) plus side-channels like the agent-flash map. Threads become another named structure in that same Y.Doc — a threads map — so they ride the document's existing sync machinery (the editor and every connected agent see new comments live, for free), but the file-writer keeps serializing only the source text. The XQ3 invariant falls out structurally rather than by discipline: the markdown file is byte-identical with or without threads, because threads simply aren't part of what gets written to it.

Durability: thread state persists under the project's internal .ok/ state directory (which is not a user-content path, so no sidecar violation). For O8 that's most likely the gitignored local area — consistent with the O4 decision that threads never travel through git. And this is what future-proofs O4: when the hosted store arrives, it syncs Y.Docs — the threads map syncs the same way the text does, and the multiplayer half becomes an audience change, not a storage rebuild.

**Non-goals:**
- [NOT NOW] Human-to-human thread delivery — that is O4, hosted track (needs O2 identity + O5 inbox).
- [NOT NOW] Thread notifications — no recipient concept locally; the thread and the timeline are the durable artifacts.
**Acceptance criteria:**
- User can anchor a thread to a text range in any doc, reply, and resolve; deleted anchor text produces an explicit orphan state, never a silent loss.
- Agents can list and read threads over MCP; an agent can reply in a thread; an agent can act on a comment and the resulting change links back to the thread (and forward into O1's jump-to-change loop).
- With threads present, the markdown files are byte-identical to a thread-free project; git push/pull behavior is unchanged.
**Assumptions:** anchor fidelity across rebinds and merges is achievable with CRDT relative positions. Confidence: **Medium** — same spike family as O1's re-anchoring assumption; shared machinery, one investigation.
**Connections:** closes a loop with O1 (comment → agent acts → traceable change you can jump to); O4 consumes this substrate for its multiplayer half; PQ5 and the parked review-gate both key off the traceability built here.

## Outcomes — hosted track (requires the federated central store)

### O2 — Every actor in a shared workspace carries a server-attested identity   `P0 · Next · [hosted]`
**Why:** All human-to-human features presuppose actors you can trust: today `principalId` and agent `clientInfo` are client-published and loopback-trusted (spoofable over a network — TQ1). Attested humans (workspace principals via inkeep identity) and attested agents (server-verified, not self-reported) are the load-bearing prerequisite for mentions, threads, notifications, and roles (platform) and for attribution users can rely on in multi-human spaces (customer trust).
**Constraints:** consumes the federated store + `app.inkeep.com` auth (spec Draft) and the member-management workstream's directory (XQ2 — Assumed); must reconcile git-derived authors with principals (TQ2); inherits locked decisions: metadata-only upstream attribution, never guess identity.
**Connections:** hard dependency of O3–O6; reuses O1's actor schema; forward: identity attestation for agents (agent-identity report's open axis).
**Promote when:** federated backend + member directory reach spec/build; then fully sharpen (auth flows, degraded states, guest questions) here first.

### O3 — A workspace member can @-mention another member, and the mentioned person reliably finds out   `P0 · Next · [hosted] `
**Why:** Mentions are the atomic unit of directing a collaborator's attention — table stakes in every comparable product (mention ⇒ notification; evidence/prior-art-collaboration.md) — and the first feature that makes OK feel multiplayer (customer) while exercising the directory contract end-to-end for the first time (platform proof of XQ2).
**Constraints:** directory = access-scoped workspace members only (no mention ⇒ no access leak); depends on O2 (attested identity) and O5 (somewhere for the notification to land); today's `@` grammar mentions documents — people-mentions must coexist with doc-mentions in one composer grammar.
**Connections:** depends on O2, O5; sibling of O4 (mentions inside comments are the highest-frequency case).
**Promote when:** O2 is in build and the directory contract (XQ2) is confirmed.

### O4 — Collaborators can discuss a doc in anchored, resolvable threads that never pollute the markdown — and agents participate in them   `P0 · Next · [hosted]`

<img src="/outcomes/human-collaboration-foundations/pasted-20260708-154205.png" />

https://liveblocks.io/docs/collaboration-features/comments

**Why:** Comments are where human-to-human collaboration actually happens on documents (table stakes: anchored threads, resolve, defined orphan behavior). OK's twist is the differentiator: threads are first-class for agents over MCP — agents can **reply in threads and act on a comment as an instruction** (make the edit, mark the thread resolved) — which no shipping prior art offers (evidence/prior-art-collaboration.md). Decided (Directed, 2026-07-08): agent participation is in scope up to *acting* (c), with *reply* (b) as the first shipped step — acting is gated on **change ↔ thread traceability, a requirement of this outcome** (an agent that edits your doc off a comment must be traceable to that comment; the trust surface and the differentiator are the same feature). This traceability lives here, not in O1 — the conversation-level "why" link was cut from O1 (PQ6). Bounded hard by storage fidelity (XQ3): anchors must survive CRDT rebinds and offline git merges, and thread storage must not appear in user markdown.

**Constraints:** XQ3 (no in-file storage, no sidecars in content paths); orphan behavior must be explicit (Docs "original content deleted" precedent); depends on O2 for authorship and O5 for reply notifications; agent *act-on-comment* traceability is owned by O8 after the split — this outcome inherits it as a given. **GitHub-sync boundary (2026-07-08):** threads never travel through git — git carries the content, the hosted store carries the conversation. Consequences: (a) git-only collaborators see clean markdown and no comments — deliberate, and an explicit trade-off to surface in sharing UX; (b) anchors must re-derive or explicitly orphan after a git-pull reconciliation the comment store never saw as CRDT ops (the XQ3 clause exists for exactly this); (c) timeline restore/rollback never deletes threads — comments stay decoupled from version history, orphaning as needed; (d) threads follow docs across git renames/moves — requires doc-identity mapping, not path binding.
**Connections:** depends on O2, O5; carries O3's mentions inside threads; agent reply/act builds on the attribution fabric O1 hardens.
**Promote when:** O2 in build. Sharpening order inside the outcome: human threads + agent read → agent reply → agent act.
**Topology decomposition (2026-07-08, user-probed):** only the multiplayer half is `[hosted]`. The thread substrate (comment store outside markdown, anchoring, resolve/orphan) and the agent loop (read/reply/act via local MCP) are `[local]`-capable today — "threads for me and my agents" needs no cloud, no other humans, and exercises this outcome's riskiest engineering (XQ3 anchor survival) pre-federation. Human-to-human delivery stays `[hosted]` (needs O2 identity + O5 inbox; git-syncing threads was considered and rejected: async arrival, merge conflicts, untrusted identity, no notification). Resolved (2026-07-08, user-directed) — carved as O8 `Now / [local]`; this outcome now owns only the multiplayer half (attested human authorship via O2, delivery + reply notifications via O5, cross-user sync of the thread store) built on O8's substrate.

### O5 — A member has one place where mentions, comment replies, and shares land   `P0 · Next · [hosted]`
**Why:** Without a recipient-side surface, mentions and comments are writes into the void — there is today no server-side representation of "another user" to hold unread state at all (evidence/current-state-collab-ux.md). One inbox with implicit subscription (Linear model) and per-trigger opt-out is the prior-art-converged shape; agent edits stay excluded by default per PQ4, which keeps the inbox human-signal-only (deliberately avoiding notification sprawl — the top prior-art failure mode).
**Constraints:** PQ4 (Directed) defines the trigger set; email/push channels are open questions for /spec; unread state lives in the hosted store (per-principal).
**Connections:** depends on O2; consumed by O3, O4, O6 (share/role grants notify).
**Promote when:** O3/O4 enter sharpening (the inbox ships with its first producer, not before).

### O7 — Workspace members can see who is in a doc right now — humans and agents alike   `P0 · Next · [hosted]`
**Why:** Today two people are never actually *on* the same doc — each edits their own copy, synced asynchronously through git; the only trace of a collaborator is post-hoc timeline attribution after a pull ([current-state-collab-ux](evidence/current-state-collab-ux.md) §6). Live presence is the moment a tool starts feeling multiplayer (Figma's presence-first model), and OK's PresenceBar + awareness fabric already render multi-actor presence — humans and agents in one bar, proven by the multi-tab path — which is the unified-fabric differentiator. What's missing is underneath the UI: a shared transport (awareness relayed through the hosted central store across machines) and attested identity to render (O2) instead of client-published names.
**Constraints:** `[hosted]` — pre-federation there is no shared live transport (loopback-only) and nothing to see; awareness relay across the hub-and-spoke topology (local server ↔ central ↔ peers) is a federated-spec /spec question; presence is access-scoped (you see only people who share access to the doc/workspace — same scoping as the mention directory). Presence is also its own *permission axis*, not a side effect of write access (Liveblocks separates `presence:write` from read/write — [captured](../../external-sources/liveblocks-room-permissions.md)): full sharpening must decide per-role presence rules — do viewers broadcast presence, and does invisible observation exist (Figma observation-mode vs the "being watched" privacy concern)? Every role×presence combination becomes an AC or a tagged non-goal.
**Connections:** depends on O2 (attested identity) + the federated transport; natural delivery pairing with O2 — presence is federation's first user-visible payoff; O3's mention directory and this outcome render the same population.
**Promote when:** O2 enters build — sharpen and present the two together.

### O9 — A member can always tell whether they're seeing live or stale state — and what happened to their offline edits   `P0 · Next · [hosted]`
**Why:** Derived from the Liveblocks sweep (2026-07-08): collaboration infrastructure treats connection/sync state as a first-class primitive, because the moment a workspace is multiplayer, *silent staleness becomes the new silent loss* — presence (O7) tells you who's here, but nothing tells you that YOU aren't; a dropped connection quietly turns a shared doc into a private fork. The house philosophy already rejects silent divergence ([internal-specs-and-reports](evidence/internal-specs-and-reports.md), silent-loss report), and every hosted comparable ships an offline/reconnecting indicator (Docs "Working offline"). Generalizes O6's offline-viewer divergence state to every role. ([prior-art-collaboration](evidence/prior-art-collaboration.md), Liveblocks addendum)
**Constraints:** visible but not naggy — a durable ambient indicator, never toasts (house philosophy); offline edits visibly queued, and reconnect reconciliation lands as attributed, durable history (O1's timeline), not a transient message; states to cover at sharpening: connected / offline-editing / reconnecting / reconciled / refused (role-based, ties to O6).
**Connections:** depends on the federated transport; complements O7 (presence = who's here; this = am I actually live); O6's viewer-divergence case is a special case of "refused"; reconciliation display rides O1's timeline fabric.
**Promote when:** O2/O7 enter build — the first hosted milestone must not introduce silent staleness.

### O6 — A workspace admin can grant view-only or comment-only access   `P2 · Later · [hosted]`
**Why:** Read/comment tiers widen who can safely be in a workspace (reviewers, stakeholders, adjacent teams) — the canonical Viewer/Commenter/Editor triad is table stakes in hosted tools, and prior art localizes real user pain at the viewer/commenter boundary. Kept open deliberately (PQ2 — user call): today "view" is served by share-a-copy, and roles only mean something on the hosted store.
**Constraints:** depends on O2 (attested identity + roles need a place to live); avoid >3-tier permission matrices at v1 (prior-art deliberately-avoid). **Git boundary (2026-07-08):** roles are enforced at the hosted sync gateway (server refuses viewer CRDT updates / commenter writes land only in the comment store) — never inside files; the local git shadow is a derived artifact of readable docs (not a bypass); a shared git remote is a parallel write path OK roles cannot govern — invariant: OK roles bind OK's live surface only, git keeps its own ACL, and git-pushed changes ingest as upstream edits under the locked attribution rules. Comment-only never conflicts with git at all (XQ3: threads have no git representation). State to sweep on promotion: viewer edits offline on a local-first client → sync refused → needs explicit divergence UX, never silent drop (silent-loss philosophy).
**Connections:** depends on O2, O5; interacts with the sharing spec's locked decisions (`/s/` reserved for cloud shares).
**Promote when:** hosted store + directory exist AND a concrete persona needs non-editor access (design partner, exec reviewer) — then decide PQ2 for real.

## Sequencing

**Now — the local track: O1 + O8** (agent change legibility; anchored threads for you + your agents). Heuristics: *dependency-first* — the local track has zero dependency on the unshipped federated backend, and its provenance + thread substrate is what O4's multiplayer threads and any future review-gate build on; *risk-first* — the pair exercises the two riskiest assumptions, which are one spike family (historical-range re-anchoring for jump-to-edit; comment-anchor survival across rebinds and git merges) while everything else waits on federation anyway; *value-first* — both are standalone product value on today's architecture, and together they close a loop: comment → agent acts → traceable change you can jump to. **Walking-skeleton test passes (user-confirmed 2026-07-08):** if federation slipped indefinitely, O1 + O8 still ship differentiated improvements. Barrel check: two parallel Now outcomes — under capacity.

**Next — O2 + O7 first, then \{O3 + O5} as one delivery group, O4 alongside/after.** O7 (live presence) ships with O2 as federation's first user-visible payoff — the moment the workspace visibly becomes multiplayer — and O9 (sync-state legibility) rides the same milestone: the first hosted release must not introduce silent staleness. O2 first (*dependency-first*: O3–O6 all consume attested identity; building any of them on client-published identity is foreclosed by TQ1). O3 and O5 must ship together (*delivery grouping*: "mention ⇒ the person finds out" is a single user promise — a mention without an inbox is a write into the void, an inbox with no producers is empty chrome). O4 rides the same identity + inbox rails on top of O8's substrate — it extends already-working threads to human recipients (authorship via O2, delivery via O5); the agent reply/act loop and its traceability ship earlier in O8 (per PQ5).

**Later — O6** (view/comment roles). Promote when the hosted store + directory exist AND a concrete non-editor persona shows up (then resolve parked PQ2 for real).

**Cross-workstream gates:** O2–O6 are blocked by (a) the federated-collaboration-sync backend (Draft spec, Andrew) and (b) the member-management workstream's directory contract (XQ2, Assumed — verify owner + contract before O2/O3 go to /spec).

## Rabbit holes

- **Building identity or membership ourselves.** Tempting because XQ2's owner is unconfirmed and O3 needs a directory; a rabbit hole because two other workstreams own it and parallel identity models are a one-way-door collision. If encountered: stop, confirm the contract with the owning workstream, keep this work consume-only.
- **Finer-than-burst attribution (per-character coloring, per-keystroke provenance).** Tempting for "who wrote this word"; the cost curve past commit/burst granularity is steep (timing report) and prior art avoids it as primary UI. If encountered: ship commit/burst granularity, revisit only on demonstrated user failure.
- **Designing comment storage/anchoring in this artifact or early /spec chatter.** Tempting because XQ3 makes it intellectually interesting; it's /spec's job. This artifact carries only the invariant (no markdown pollution, anchors survive merges).
- **Notification preference matrices.** Tempting completeness ("per-doc, per-actor, per-event settings"); prior art's failure mode is sprawl. Ship one inbox + implicit subscription + per-trigger opt-out, nothing more at v1.
- **Pending-review gating for agent edits.** Explicitly parked [NOT UNLESS]; if it resurfaces mid-implementation, check the trigger (agents editing *others'* docs) before reopening.

## Pre-mortem

Most likely failure: **the federated backend slips and "cloud collaboration" stalls at O1** — expectation set as multiplayer, delivered as better agent legibility. Mitigation: topology tags make the gate explicit; O1 is real standalone value; O2–O5 sharpening waits for promotion triggers instead of speccing against a moving backend.
Second: **the directory-contract assumption (XQ2, Medium) is wrong** — the member-management workstream delivers something mentions/roles can't consume (wrong scoping, no avatars, no query surface), forcing O3/O6 rework. Mitigation: named verification plan — confirm owner + contract before /spec.
Third: **agent act-on-comment erodes trust** if it ships before change↔thread traceability is airtight — one bad unattributable agent edit off a comment poisons the differentiator. Mitigation: reply-before-act ordering is recorded as Directed, and traceability is an O4-owned requirement.

## Evidence & References

### Evidence Files
- [current-state-collab-ux](evidence/current-state-collab-ux.md) — codebase reality: identity, presence, attribution fabric, timeline, sharing, gaps (raw proof, file:line)
- [internal-specs-and-reports](evidence/internal-specs-and-reports.md) — federated/timeline/sharing spec constraints, locked decisions, house philosophy from reports
- [prior-art-collaboration](evidence/prior-art-collaboration.md) — Notion/Docs/Figma/Obsidian-Relay/Linear + AI-editor synthesis: table stakes / differentiators / deliberately avoid; incl. the Notion Updates-feed deep-linking addendum

### Upstream Artifacts
- [Federated Collaboration Sync spec — captured summary](../../external-sources/federated-collaboration-sync-spec.md) — Draft spec (owner Andrew) modeling hosted multi-tenant sync + `TENANT → PRINCIPAL` identity (engine repo: `specs/2026-05-20-federated-collaboration-sync/SPEC.md`)
- Monorepo companion artifact: `outcomes/ok-collaboration-experience/OUTCOMES.md` in the engine repo (agents-private) — kept in sync with this doc; carries the session changelog in `meta/_changelog.md`

