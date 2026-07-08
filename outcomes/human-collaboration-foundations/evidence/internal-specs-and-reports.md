---
title: OK collaboration UX — internal specs & reports extraction (channel 2)
description: "What the engine repo's internal specs and reports lock or constrain for collaboration UX: federated-collaboration-sync identity model (proposed, not confirmed), timeline upstream-author attribution decisions, sharing-spec locked decisions, and house philosophy from the silent-loss / timing / agent-follow reports."
tags:
  - evidence
  - collaboration
  - phase-2
  - raw-proof
type: raw-proof
created: 2026-07-08
---
# Internal specs & reports that constrain the collaboration-experience work

Sources are engine-repo paths (inkeep agents-private monorepo). The federated spec's captured summary also lives locally: [Federated Collaboration Sync spec (captured)](../../../external-sources/federated-collaboration-sync-spec.md).

## A. Federated Collaboration Sync spec (2026-05-20, Draft, owner Andrew)

`public/open-knowledge/specs/2026-05-20-federated-collaboration-sync/SPEC.md`

- **CONFIRMED — Identity model (proposed, not confirmed):** ER diagram (SPEC lines 133-169): `TENANT ||--o{ PRINCIPAL` ("has members"), `TENANT ||--o{ WORKSPACE`, `WORKSPACE ||--o{ DOCUMENT`, `PRINCIPAL ||--o{ SESSION`. `PRINCIPAL { id, tenant_id FK, string inkeep_identity }` — i.e., every principal carries an `inkeep_identity` mapping to `app.inkeep.com` auth. Note at line 171: hierarchy + `content_namespace` mapping are "proposed, not confirmed"; per-user vs per-org tenancy is an open product question (Q3).
- **CONFIRMED — Locked decisions (D1-D5, all DIRECTED):** production-grade hosted service; local-first hub-and-spoke (local server ↔ Inkeep-hosted central store), P2P deferred; mobile first-class, connecting directly to central; central store is a *capable* server (bridge + persistence + auth + tenancy), 1-way door; auth reuses `app.inkeep.com` mapped onto OK's principal model (1-way door; token shape open Q4).
- **CONFIRMED — Open questions Q1-Q6 (all P0/blocking):** server↔server Y.js sync primitive; bridge locality; tenancy unit + docName namespace; token shape → principal mapping; central durability; conflict/authority with multiple local servers on one doc.
- **CONFIRMED — Current-state grounding (§8):** "No authentication on the transport... missing/invalid tokens silently ignored"; "single-instance, single-tenant"; "host/port already configurable — binding remotely is not the blocker."
- **SUPPORTED — What the spec does NOT cover:** collaboration *UX* (mentions, comments, view-only, notifications) is entirely absent — the spec is transport/tenancy/auth-focused. §4 Personas and §5 Journeys are unfilled scaffolds. So the outcomes-shaping session on collaboration experience has no upstream UX decisions to inherit; only the identity substrate (TENANT→PRINCIPAL with inkeep_identity) and the topology constraints bound it.
- Evidence dir contains `_user_outcomes.md` (verbatim user direction: "production grade remote collaboration setup", mobile yes, reuse app.inkeep.com auth) and `current-state-server.md`.

## B. Timeline upstream-author attribution spec (2026-07-06, Draft, owner Shagun Singh — engine repo root)

`specs/2026-07-06-timeline-upstream-author-attribution/SPEC.md`

- **CONFIRMED — Problem:** teammate changes arriving via `git pull` were all attributed "File System" in the Timeline; the user had to go to GitHub to see who changed what (§1, real reported session with `amikofalvy`, `nick-inkeep`).
- **CONFIRMED — Chosen approach (Strategy A):** metadata-only attribution — recover the real commit author (name + email) per doc from the `oldHead..newHead` range at HEAD-move drain time; display name comes from commit metadata (`contributors[].name`) while the writer bucket stays `git-upstream`. No taxonomy change (G2/NFR1). Non-attributable disk writes stay `file-system` (G3, D3: "never guess identity").
- **CONFIRMED — Decisions:** D2 last non-merge author wins per file; D3 fall back to file-system on any ambiguity; D4 remote authors with no local principal render as their git name with an email-seeded color.
- **CONFIRMED — Deliberately deferred (NG1/Strategy B):** per-author writer refs (`git-author-<hash>`) — higher fidelity per-author WIP refs; forces taxonomy/GC changes. NG3: no backfill of past `file-system` entries.
- **CONFIRMED — Open Qs:** Q1 multi-author-per-pull display (default last-author-wins); Q2 whether the `git-upstream` boundary commit subject names authors (default: neutral).
- **SUPPORTED — Implementation appears landed (at least partially):** `resolveUpstreamChanges` exists in `packages/server/src/server-factory.ts` with unit tests `resolve-upstream-author.test.ts` whose docblock matches the spec's FR1 exactly.
- **Relevance to collaboration UX:** this is the *only* existing mechanism by which a remote human collaborator's identity surfaces inside OK — and it is post-hoc, git-derived, display-name-only (no principal, no mentionability, no avatar identity beyond an email-seeded color).

## C. Sharing & Virality Flow spec (2026-05-14, Approved, owner Miles K-T)

`public/open-knowledge/specs/2026-05-14-sharing-virality-flow/SPEC.md`

- **CONFIRMED — Sharing model:** share = marketing-safe URL `openknowledge.ai/d/<base64url(github-blob-url)>`; splash page; receiver *receives a copy* into their own OK (clone / open-local / already-known project). No hosted backend, no share-id store (NG1 NOT NOW); collaboration continues over git.
- **CONFIRMED — Identity implications:** D31/FR8 — **no sender attribution** on the splash page ("receivers infer the sender from the message context"); D21 forbids cross-user identifiers in URLs (killed a latency metric). Share is read-only (FR17); publish is an explicit human wizard; auto-commit-on-share removed by user directive (NG11).
- **CONFIRMED — Forward-fit:** URL path prefixes reserved for future share types — `/s/<cloud-share-id>` for OK-native cloud shares (D15) — an explicit hook for the federated/hosted future.
- **Relevance:** establishes the house philosophy that (pre-federation) "collaborator" = someone you route through GitHub, identity = GitHub identity, and OK deliberately avoids storing cross-user identifiers.

## D. Report: agent-identity-attribution-worldmodel (baseline 2026-04-18 — PARTIALLY STALE)

`public/open-knowledge/reports/agent-identity-attribution-worldmodel/REPORT.md`

- **CONFIRMED (as of its baseline) — Identity topology:** agent identity is per-MCP-subprocess (`connectionId = randomUUID()` at start), N subprocesses per project, one Hocuspocus server per contentDir; identity flows MCP handshake → HTTP body → `extractAgentIdentity` → session keyed `(docName, agentId)` → awareness + shared `AGENT_WRITE_ORIGIN` → contributor-tracker → shadow WIP commits.
- **CONFIRMED — Gap catalog (§12, deep verification):** at that baseline, rename/rollback MCP tools did not thread identity; committer hardcoded `openknowledge`; ghost agents lingered in PresenceBar; `agentFocus` never cleared; default identity bucket `claude-1` for non-MCP callers; branch park collapsed per-agent state; checkpoints/rollbacks invisible to attribution.
- **STALENESS NOTE (verified against current code):** several §12 gaps have since been fixed — `extractActorIdentity` now covers rename/rollback, the attribution-sweep meta-test enforces per-handler identity, keepalive-WS close now clears agent presence, and `agentPresence`/`agentFocus` broadcasting landed. Treat §12 as a historical gap map, not current truth; §6/§9 open framings (per-agent undo origin, cross-project identity, identity attestation/trust — "if MCP clientInfo lies, what breaks") remain live questions.
- **CONFIRMED — 3P observation:** "No industry convention surfaced for (a) per-process vs per-agent-type granularity, (b) agent/human distinction in awareness payload, (c) identity chain unifying MCP + awareness + git."

## E. Report: collab-editor-silent-loss-ux-patterns (2026-04-16)

`public/open-knowledge/reports/collab-editor-silent-loss-ux-patterns/REPORT.md`

- **CONFIRMED — TLDR:** for merge anomalies that may lose content, the industry pattern is a *silent recoverable artifact* (Notion duplicate page, Docs version history), not toasts; "toasts carry trust-erosion cost." No production editor toasts mid-typing merge loss. Obsidian's silent-loss-without-signal produced a 4-year-open complaint thread — silence is only fine when merge is provably lossless.
- **Relevance:** sets the house notification philosophy — prefer durable artifacts surfaced in the Timeline over interruptive notifications.

## F. Report: collaborative-editor-timing-best-practices (2026-04-16)

`public/open-knowledge/reports/collaborative-editor-timing-best-practices/REPORT.md`

- **CONFIRMED — TLDR:** perception tiers (<100ms instantaneous / 100-300ms responsive / 300-1000ms noticeable); OK's constants validated against ecosystem (bridge 50ms, persistence 2s/10s = Hocuspocus default, git commit debounce 15s ≈ VS Code local history 10s / Figma checkpoints 30-60s, CC1 100ms = Liveblocks throttle).
- **Relevance:** bounds attribution granularity — L2 commit debounce (~15s, cross-doc batched) is the floor of Timeline-entry granularity; anything finer (per-burst) lives in the Agent Activity Panel's ring buffer, not git.

## G. Report: agent-follow-and-edit-visibility-ux (2026-04-20)

`public/open-knowledge/reports/agent-follow-and-edit-visibility-ux/REPORT.md`

- **CONFIRMED — Headline:** "pin an agent, follow them across files, glow their presence, expose edits as a diff timeline" has **no shipping prior art combining all four**; pieces exist (Replit filetree presence, Live Share pushpin follow, Cursor inline diff, Devin session replay, Notion avatar pulse).
- **CONFIRMED — Recommendations:** explicit-only follow-break (Live Share model, not Docs auto-break); Replit filetree-avatar for cross-file presence; AI inline diff (Cursor/Zed) as primary edit visualization, not live cursors; two feed shapes (per-agent scrubber + workspace reverse-chron); design multi-agent from day one.
- **Relevance:** OK has since shipped much of this (agentPresence + activity panel + timeline); the un-shipped novel affordances it flagged (pause-the-agent during follow, rewind scrubbing, per-file follow filter) remain open differentiators.

## H. Reports-dir scan (negative + adjacent)

Scanned the engine repo's `reports/` (~300 dirs) for collaboration/identity/notification-relevant titles. Adjacent but not read in full: `mcp-agent-attribution-implementation`, `timeline-scope-filter-patterns`, `auto-persistence-version-history-patterns`, `in-chat-diff-rendering-precedents`, `crdt-branching-namespacing-prior-art`, `sharing-virality-flow-design`, `open-from-github-onboarding-mechanics`, `okf-adoption-signal-2026-06-18`. **NOT FOUND:** no report on comments/threads, people-@-mentions, view-only access, or a notification model — searched report dir names for `comment|mention|notif|permission|access` → zero relevant hits. This confirms human-to-human collaboration UX is unexplored territory internally.
