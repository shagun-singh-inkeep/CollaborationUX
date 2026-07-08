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
  1. **Collaborator identity** — who can you @-mention; leaning toward "Option 4 — Federated / OK-native accounts" (identity lives in OK's hosted backend; workspace membership = directory; the Notion/Figma model), which is already modeled in the Draft [federated-collaboration-sync spec](../../public/open-knowledge/specs/2026-05-20-federated-collaboration-sync/SPEC.md) as `TENANT → PRINCIPAL` with an `inkeep_identity` per principal.
  2. **Comments and threads.**
  3. **View-only** — open question whether it should exist at all ("would that be a thing??").
  4. **Some sort of notification model.**

**Strategic intake (Shagun, 2026-07-08):**
- **Why now:** Inkeep is starting to think about the *cloud version* of OK; better collaboration is part of that push. This work is UX-shaping for the cloud direction.
- **Sequencing vs. federation:** undecided. The federated-collaboration-sync backend has NOT shipped. Open question whether any of this lands on today's local-first/no-auth architecture or all of it assumes the hosted backend.
- **Appetite:** open-ended (shaping, no time box).
- **Adjacent work in flight:** member invites, account and team management are owned by someone else. This work must *consume* workspace membership/identity, not build or re-spec it.
- **Kill risk:** not yet articulated (pre-mortem to develop it).

## Items

| ID | Item | Type | Priority | Status | Notes |
|---|---|---|---|---|---|
| XQ1 | Topology tagging: which outcomes land on today's local-first/git architecture vs the federated hosted store? | Cross-cutting | P0 | Decided | **Directed (2026-07-08):** every outcome carries a `[local]` / `[hosted]` / `[both]` tag. Agent-legibility work is buildable today; all human-to-human primitives presuppose the hosted store (no recipient concept pre-federation). (evidence/current-state-collab-ux.md) |
| XQ2 | Interface with the member-invites / account & team management workstream | Cross-cutting | P0 | Assumed | **Claim:** that workstream delivers a queryable, access-scoped member directory (names, avatars, principal IDs) backed by app.inkeep.com identities, consumable by mentions/comments/notifications. Confidence: **Medium** (auth reuse is locked in the federated spec; the directory *contract* is unconfirmed — user doesn't know the owner/status). **Verify by:** identify the workstream owner and confirm the directory contract before /spec of O2/O3. |
| PQ1 | Collaborator identity model — Option 4 (federated / OK-native accounts) | Product | P0 | Assumed | **Claim:** identity will be OK-native/federated per the federated spec's TENANT → PRINCIPAL / inkeep_identity direction (user's lean; spec is Draft — model proposed, not confirmed). Treated as inherited direction + dependency, NOT this artifact's decision. Confidence: **Medium**. **Verify by:** confirm with federated-spec owner (Andrew) that the principal model survives spec finalization; revisit O2/O3 requirements if it changes. |
| PQ2 | Should view-only access exist? | Product | P0 | Parked | **User call (2026-07-08): keep open.** Reframe recorded: "view-only" = (a) share-a-copy (exists), (b) live view of a local server (needs auth), (c) hosted-store role. Lean: scope as hosted-store role with a *commenter* middle tier from day one (prior art: pain concentrates at viewer/commenter boundary). **Trigger to revisit:** hosted store + member directory exist, or O6 gets promoted. (evidence/prior-art-collaboration.md) |
| TQ1 | Server-authoritative actor attribution on the transport | Technical | P0 | Decided | **Directed (2026-07-08):** resolved by promotion to outcome O2 — every networked feature depends on it; no h2h outcome ships on client-published identity. (evidence/current-state-collab-ux.md) |
| TQ2 | Identity reconciliation: git-derived identity ↔ inkeep principal | Technical | P0 | Decided | **Delegated (2026-07-08):** the requirement is locked (one human = one rendered identity across timeline + directory; inherit "never guess, metadata-only"); the reconciliation design belongs to /spec of O2/O3. |
| PQ5 | How far do agents go in comment threads: read / reply / act? | Product | P0 | Decided | **Directed (2026-07-08, Shagun): "(c) or (b)"** — agent participation in scope up to acting on comments; ship order reply → act, act gated on change↔thread traceability (owned by O4). |
| PQ6 | Does O1 need a "why" link from a change to its originating conversation/task? | Product | P0 | Decided | **Directed (2026-07-08, Shagun): no** — cut from O1 scope; existing write summaries suffice as the "why". Demoted to [NOT NOW] non-goal on O1 with revisit triggers (summaries prove insufficient, or O4 act-on-comment lands). Side effect: kills O1's riskiest assumption (MCP conversation-reference capture). |
| PQ3 | Agent-edit trust: observability vs pending-review gating | Product | P0 | Decided | **Directed (2026-07-08): Option A — observability, no gating.** Double down on flash + presence + timeline + per-agent undo. Pending-review parked as **[NOT UNLESS]** — condition: multi-human workspaces where one member's agent edits other people's docs unreviewed (the calculus changes there; prior art says adding review later is safe, removing it isn't). User also added a sharpening requirement → O1: timeline entries must *jump to the edited part in the doc*, not only show a diff. |
| PQ4 | Notification philosophy: what notifies? | Product | P0 | Decided | **Directed (2026-07-08, no objection to stated intention):** mentions / comment replies / shares notify into ONE inbox with implicit subscription (Linear model); agent edits never notify by default — the timeline is their durable artifact (house philosophy). |
| XQ3 | Comment storage must not pollute the markdown files | Cross-cutting | P0 | Decided | **Locked as invariant (2026-07-08):** thread/notification metadata never appears in user markdown or as sidecars in content paths (storage-fidelity contract); anchors must survive CRDT rebinds and offline git merges. The storage *design* is Delegated to /spec of O4/O5. |

## Cross-cutting concerns

- **Topology tags (XQ1)** — every outcome is tagged `[local]` / `[hosted]` / `[both]`; hosted-tagged outcomes are blocked by the federated-collaboration-sync backend shipping. Touches all outcomes. Owner: this artifact.
- **Member directory & identity (XQ2, PQ1, TQ2)** — mentions, comments, notifications, and roles all consume one access-scoped directory of workspace principals (inkeep-identity-backed), reconciled with git-derived authors in history. Touches O2–O6. Owner: member-management workstream + federated spec (Andrew); this work states requirements only.
- **Trusted attribution on the transport (TQ1)** — client-published identity is loopback-trusted today; every networked feature requires server-attested actors first. Touches O2–O6 (and O1 under federation). Owner: federated spec; surfaced here as O2.
- **Storage fidelity (XQ3)** — no sidecars in user-content paths; markdown files must stay byte-faithful; collaboration metadata (threads, notification state) needs a managed store whose anchors survive CRDT rebinds and offline git merges. Touches O4, O5. Owner: /spec, bounded here.
- **Notification philosophy (PQ4)** — one inbox; mentions/comments/shares notify with implicit subscription; agent edits never notify by default (timeline is their durable artifact). Touches O3, O4, O5, O6.

## Outcomes

### O1 — A human working alongside agents can trace any agent change to *what and who* — and jump to it in the doc   `P0 · Now · [both]`
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
- [NOT NOW] Deep link from a change back to the originating conversation/task (PQ6 — user-directed cut; also killed this outcome's riskiest assumption). Revisit if: write summaries prove insufficient to answer "why did the agent do this" in real usage, or when O4's act-on-comment lands (where the thread itself is the why).
**Acceptance criteria:**
- From any timeline or activity-panel entry, one action scrolls the current doc to the corresponding region with an in-context highlight; deleted regions get an explicit "content no longer present" affordance (Docs orphaned-comment precedent) rather than a silent no-op.
- User can step prev/next through the changes of an agent session in-document (Docs "see new changes" / Word track-changes navigation pattern), not only read a diff pane.
- Every agent change durably exposes what (diff), who (agent + session + principal), and when — with the existing agent-supplied write summary rendered when present (no new "why" machinery).
- A user returning after hours away can reconstruct "what did agents do while I was gone" from durable UI alone — no reliance on toasts or having watched live.
**Assumptions:**
- Historical change ranges can be mapped to current doc positions (CRDT relative positions / diff re-anchoring) with acceptable fidelity. Confidence: **Medium**. Verify by: spike on the existing timeline diff data. Note: Notion proves the end-to-end UX at scale via stable block-UUID anchors (`#block-uuid` deep links from its Updates feed — evidence/prior-art-collaboration.md addendum); OK's engineering delta is deriving equivalent durable anchors without a block model — CRDT relative positions are the natural analog.
**Connections:** traces to cloud-version push; extends `specs/2026-07-06-timeline-upstream-author-attribution` and the existing attribution fabric (precedent #25); O2 reuses the actor schema for humans; O4 (agents in threads) and the parked review-gate would build on the same durable provenance.

### O2 — Every actor in a shared workspace carries a server-attested identity   `P0 · Next · [hosted]`
**Why:** All human-to-human features presuppose actors you can trust: today `principalId` and agent `clientInfo` are client-published and loopback-trusted (spoofable over a network — TQ1). Attested humans (workspace principals via inkeep identity) and attested agents (server-verified, not self-reported) are the load-bearing prerequisite for mentions, threads, notifications, and roles (platform) and for attribution users can rely on in multi-human spaces (customer trust).
**Constraints:** consumes the federated store + `app.inkeep.com` auth (spec Draft) and the member-management workstream's directory (XQ2 — Assumed); must reconcile git-derived authors with principals (TQ2); inherits locked decisions: metadata-only upstream attribution, never guess identity.
**Connections:** hard dependency of O3–O6; reuses O1's actor schema; forward: identity attestation for agents (agent-identity report's open axis).
**Promote when:** federated backend + member directory reach spec/build; then fully sharpen (auth flows, degraded states, guest questions) here first.

### O3 — A workspace member can @-mention another member, and the mentioned person reliably finds out   `P0 · Next · [hosted]`
**Why:** Mentions are the atomic unit of directing a collaborator's attention — table stakes in every comparable product (mention ⇒ notification; evidence/prior-art-collaboration.md) — and the first feature that makes OK feel multiplayer (customer) while exercising the directory contract end-to-end for the first time (platform proof of XQ2).
**Constraints:** directory = access-scoped workspace members only (no mention ⇒ no access leak); depends on O2 (attested identity) and O5 (somewhere for the notification to land); today's `@` grammar mentions documents — people-mentions must coexist with doc-mentions in one composer grammar.
**Connections:** depends on O2, O5; sibling of O4 (mentions inside comments are the highest-frequency case).
**Promote when:** O2 is in build and the directory contract (XQ2) is confirmed.

### O4 — Collaborators can discuss a doc in anchored, resolvable threads that never pollute the markdown — and agents participate in them   `P0 · Next · [hosted]`
**Why:** Comments are where human-to-human collaboration actually happens on documents (table stakes: anchored threads, resolve, defined orphan behavior). OK's twist is the differentiator: threads are first-class for agents over MCP — agents can **reply in threads and act on a comment as an instruction** (make the edit, mark the thread resolved) — which no shipping prior art offers (evidence/prior-art-collaboration.md). Decided (Directed, 2026-07-08): agent participation is in scope up to *acting* (c), with *reply* (b) as the first shipped step — acting is gated on **change ↔ thread traceability, a requirement of this outcome** (an agent that edits your doc off a comment must be traceable to that comment; the trust surface and the differentiator are the same feature). This traceability lives here, not in O1 — the conversation-level "why" link was cut from O1 (PQ6). Bounded hard by storage fidelity (XQ3): anchors must survive CRDT rebinds and offline git merges, and thread storage must not appear in user markdown.
**Constraints:** XQ3 (no in-file storage, no sidecars in content paths); orphan behavior must be explicit (Docs "original content deleted" precedent); depends on O2 for authorship and O5 for reply notifications; agent *act-on-comment* requires change ↔ thread traceability (owned by this outcome).
**Connections:** depends on O2, O5; carries O3's mentions inside threads; agent reply/act builds on the attribution fabric O1 hardens.
**Promote when:** O2 in build. Sharpening order inside the outcome: human threads + agent read → agent reply → agent act.

### O5 — A member has one place where mentions, comment replies, and shares land   `P0 · Next · [hosted]`
**Why:** Without a recipient-side surface, mentions and comments are writes into the void — there is today no server-side representation of "another user" to hold unread state at all (evidence/current-state-collab-ux.md). One inbox with implicit subscription (Linear model) and per-trigger opt-out is the prior-art-converged shape; agent edits stay excluded by default per PQ4, which keeps the inbox human-signal-only (deliberately avoiding notification sprawl — the top prior-art failure mode).
**Constraints:** PQ4 (Directed) defines the trigger set; email/push channels are open questions for /spec; unread state lives in the hosted store (per-principal).
**Connections:** depends on O2; consumed by O3, O4, O6 (share/role grants notify).
**Promote when:** O3/O4 enter sharpening (the inbox ships with its first producer, not before).

### O6 — A workspace admin can grant view-only or comment-only access   `P2 · Later · [hosted]`
**Why:** Read/comment tiers widen who can safely be in a workspace (reviewers, stakeholders, adjacent teams) — the canonical Viewer/Commenter/Editor triad is table stakes in hosted tools, and prior art localizes real user pain at the viewer/commenter boundary. Kept open deliberately (PQ2 — user call): today "view" is served by share-a-copy, and roles only mean something on the hosted store.
**Constraints:** depends on O2 (attested identity + roles need a place to live); avoid >3-tier permission matrices at v1 (prior-art deliberately-avoid).
**Connections:** depends on O2, O5; interacts with the sharing spec's locked decisions (`/s/` reserved for cloud shares).
**Promote when:** hosted store + directory exist AND a concrete persona needs non-editor access (design partner, exec reviewer) — then decide PQ2 for real.

## Sequencing

**Now — O1** (agent change legibility). Heuristics: *dependency-first* — it is the only outcome with zero dependency on the unshipped federated backend, and its durable provenance is what O4's agent-act and any future review-gate build on; *risk-first* — it exercises the remaining risky assumption (historical-range re-anchoring for jump-to-edit) while everything else waits on federation anyway; *value-first* — it is standalone product value on the architecture users run today. **Walking-skeleton test passes (user-confirmed 2026-07-08):** if federation slipped indefinitely, O1 alone still ships a differentiated improvement. Barrel check: one parallel Now outcome — well under capacity.

**Next — O2, then {O3 + O5} as one delivery group, O4 alongside/after.** O2 first (*dependency-first*: O3–O6 all consume attested identity; building any of them on client-published identity is foreclosed by TQ1). O3 and O5 must ship together (*delivery grouping*: "mention ⇒ the person finds out" is a single user promise — a mention without an inbox is a write into the void, an inbox with no producers is empty chrome). O4 rides the same identity + inbox rails; its internal order is human threads → agent reply → agent act (act gated on O1 provenance, per PQ5).

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
- [evidence/current-state-collab-ux.md](evidence/current-state-collab-ux.md) — codebase reality: identity, presence, attribution fabric, timeline, sharing, gaps (raw proof, file:line)
- [evidence/internal-specs-and-reports.md](evidence/internal-specs-and-reports.md) — federated/timeline/sharing spec constraints, locked decisions, house philosophy from reports
- [evidence/prior-art-collaboration.md](evidence/prior-art-collaboration.md) — Notion/Figma/Docs/Obsidian/Linear + AI-editor synthesis: table stakes / differentiators / deliberately avoid

### Upstream Artifacts
- [federated-collaboration-sync SPEC.md](../../public/open-knowledge/specs/2026-05-20-federated-collaboration-sync/SPEC.md) — Draft spec (owner Andrew) modeling hosted multi-tenant sync + `TENANT → PRINCIPAL` identity

