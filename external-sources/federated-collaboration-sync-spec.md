---
title: Federated Collaboration Sync spec (Inkeep-hosted central store) — captured summary (source)
description: "Captured summary of the engine-repo spec 2026-05-20-federated-collaboration-sync: an Inkeep-hosted, multi-tenant central store for real-time multi-user OK collaboration; local-first preserved; mobile thin clients; auth reuses app.inkeep.com mapped to the principal model. Draft; owner Andrew."
type: source
source_repo: inkeep/open-knowledge (engine)
source_path: specs/2026-05-20-federated-collaboration-sync/SPEC.md
source_status: Draft
source_owner: Andrew
fetched: 2026-07-06
tags:
  - source
  - federated
  - cloud
  - collaboration
  - identity
  - phase-2
---
# Federated Collaboration Sync spec (Inkeep-hosted central store) — captured summary (source)

> Captured 2026-07-06 from the engine repo (`inkeep/open-knowledge`) at `specs/2026-05-20-federated-collaboration-sync/SPEC.md` — **Status: Draft**, owner **Andrew**. This is the canonical **cloud / hosted collaboration** spec that OK's [[specs/human-to-human-collaboration/spec|Phase 2 work]] is gated on; summarized here because the spec itself lives outside this knowledge base. Preserved to ground [[guides/collaborator-identity|the collaborator identity research]] and the [[guides/collaboration-status|build status]].

## What it is

An Inkeep-hosted, **multi-tenant central store** for production-grade real-time collaboration, keeping OK **local-first**. Two client classes federate through one central server:

- **Class A — server-backed (desktop / dev):** client → local OK server → central store. Files on disk + git shadow repo; offline by default; the local server runs the bridge.
- **Class B — thin (mobile / browser-only):** client → central store directly. No local server; offline-capable via client-side Y.js (IndexedDB). The central server runs the bridge on its behalf.

The central server is **capable** (bridge + persistence + auth + tenancy), not a passive relay.

## Goals (G1–G6)

- **G1** — multiple authenticated users collaborate in real time via the central store.
- **G2** — local-first preserved; offline edits converge on reconnect.
- **G3** — mobile / thin clients connect directly, offline-capable.
- **G4** — tenant isolation.
- **G5** — **authentication reuses `app.inkeep.com`; identity maps onto OK's principal model.**
- **G6** — production-grade: durable storage, observability, security, scaling.

## Current state it confirms

Matches what [[guides/collaborator-identity|the identity research]] found first-hand: today OK is **filesystem + git, no database**; **no authentication on the collaboration transport**; single-tenant, one local server per content directory. Remote binding is already easy; auth, tenancy, a server↔server sync primitive, durable hosted storage, and a multi-instance story are all **net-new**.

## Identity model — decided vs open

Entity model: `TENANT → WORKSPACE → DOCUMENT`; a `TENANT` has member `PRINCIPAL`s; each `PRINCIPAL` carries an `inkeep_identity` and is authenticated by the auth server.

- **Decided (1-way-door):** **D5 — auth reuses `app.inkeep.com`, mapped to the OK principal model.** **D4 — the central store owns auth + tenancy** (a capable server, not a relay).
- **Open (P0, blocking):** **Q4 — the `app.inkeep.com` token shape and how it maps to the OK principal.** **Q3 — the tenancy unit (per-user vs per-org workspace) and the `documentName → tenant` namespace.**
- **Not addressed at all:** the mention **directory** ("who can I `@`?") and mention / notification resolution. The spec sets the auth *direction* but not the people-directory — which is the gap [[guides/collaborator-identity|the identity research]] works through.

## Open questions (all P0, blocking)

- **Q1** — server-to-server Y.js sync primitive (Hocuspocus-as-client, y-sweet, y-redis, custom relay).
- **Q2** — where the dual-CRDT bridge runs per client class; whether cross-server merge residual stays bounded.
- **Q3** — tenancy unit + namespace mapping.
- **Q4** — `app.inkeep.com` token shape → OK principal mapping.
- **Q5** — central-store durability / backup model (filesystem + git today).
- **Q6** — authority when multiple local servers + direct clients edit one document.

## Why it matters here

This is the concrete document behind [[guides/view-only-access|view-only]] open question #4 and the identity research's **Option 4**. It means "federated accounts" is **not hypothetical and not a greenfield database** — it reuses Inkeep's existing `app.inkeep.com` auth — but it is **Draft with every identity sub-question (Q3/Q4) still open**, and it says nothing about the mention directory.
