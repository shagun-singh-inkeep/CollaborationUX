---
type: proposal
title: Contribute from OK — fork-and-propose (in-product PR) flow
description: Let a user without push access contribute their edits back to the source repo from inside OK — an in-product “propose changes / open PR” action — turning today's dead-end local copy into a real fork-and-contribute loop.
status: draft
authors:
  - shagun@inkeep.com
created: 2026-07-06
tags:
  - proposal
  - collaboration
  - github
  - pull-request
  - contribution
  - phase-2
---
# Proposal 0002 — Contribute from OK (fork-and-propose / in-product PR)

**Status:** draft · **Author:** shagun@inkeep.com · **Created:** 2026-07-06

Parent hub: [[README|Collaboration UX]]. Related: [[proposals/0001-collaboration-ux|Proposal 0001]] (open question #4) · [[guides/view-only-access|view-only access research]] · [[guides/collaboration-status|build status]].

## Motivation

**OK has no way to contribute edits back to a repo you can't push to.** Verified in the `inkeep/open-knowledge` code: the only GitHub write OK performs is `repos.createForAuthenticatedUser` / `createInOrg` in the share/publish flow (`publish.ts`) — it creates a **new repo** and pushes your copy there. There is **no `pulls.create` / open-a-PR path anywhere** in the product. Sync (`sync-engine.ts`) only pushes commits directly to your remote's branch; if you lack push permission, sync just **pauses** with a toast (`github-permissions.ts`) and your commits pile up locally.

So a `pull`-only user today gets a **dead-end copy** — they can edit their clone, but the only ways back to the original are to leave OK and open a PR on GitHub by hand, or to publish an unlinked new repo. This is the same dead end Google Docs' *Make a copy* has, and it undercuts what should be OK's git-native advantage.

This matters more than it first looks. The [[guides/view-only-access|view-only research]] concluded that OK **mostly doesn't need a permission system** — GitHub already gates who can change the canonical doc (`push` vs `pull`). The missing half of that story is the *way back*: a `pull`-only user needs to be able to **propose** changes, not just be blocked. **Building that contribute-back loop is plausibly higher-value than any view-only enforcement**, and it's the natural home for a future suggestion/review flow.

## Design

Add a first-class **“Propose changes”** action that opens (or updates) a pull request against the source repo, from inside OK.

### The flow

1. **Detect the context.** Reuse the `permissions.push` signal OK already fetches (`github-permissions.ts`): no push → the user is a contributor, not a direct editor.
2. **Land the edits on a pushable branch.** Two variants by access:
   - **Has push, wants review** → push a feature branch to the same repo and open a PR (`base = default branch`, `head = branch`).
   - **No push (`pull`-only)** → **fork** the repo to the user's account (Octokit `repos.createFork` — not currently used), push the branch to the fork, and open a **cross-fork PR** (`head = user:branch`).
3. **Open the PR.** Octokit `pulls.create` against the upstream — the one API call OK doesn't make today. Prefill the PR **title/body from the change summaries OK already generates** (the Timeline / `agent-effects` data), so the proposal is legible without extra typing.
4. **Reflect state in the UI.** Replace today's silent “edit → sync paused” confusion with a legible fork state: *“you're editing your own copy — Propose changes to send them upstream,”* then show PR status (open / merged / changes-requested) once created.

### Reuse, don't rebuild

- **Auth + Octokit** are already wired (`@octokit/rest`, device-flow / PAT auth in `packages/cli/src/auth`, `publish.ts`). This adds `repos.createFork` + `pulls.create` to an existing client, not a new integration.
- **Change summaries** for the PR body already exist (the [[specs/agent-change-visibility/spec|Phase 1]] Timeline data).
- **Branch/worktree awareness** already exists (`git-branch-info.ts`, worktree selectors), so “put these edits on a branch” is not greenfield.

### Scope

v1 = the round-trip: fork (if needed) → branch → push → `pulls.create` → show status. Not in v1: in-editor suggestion mode (a separate, larger surface — see [[specs/human-to-human-collaboration/spec|Phase 2]]), non-GitHub remotes, and PR *review* inside OK.

## Drawbacks

- **Fork lifecycle.** Cross-fork PRs mean OK now manages a user fork (create-if-missing, keep-in-sync, stale forks). Real complexity, though GitHub owns the hard parts.
- **Auth scope.** Forking + PR needs a token scope beyond read; `pull`-only users may need to re-consent. Must be legible, not a silent failure.
- **GitHub-only.** The design assumes a GitHub remote. Non-GitHub git hosts (or the future federated store) need a different path or a graceful “not available here.”
- **Overlap with federated real-time.** For live shared sessions the answer may instead be auto-fork at the write path (see [[guides/view-only-access|view-only research]]); this proposal targets the **async / git** path. The two shouldn't be designed in conflict.

## Alternatives

- **Status quo — manual PR on GitHub.** Works, but the user leaves OK and the local-first edits have to be reconciled by hand. High friction; the capability is invisible.
- **Publish-as-new-repo (today's share).** Already exists (`publish.ts`) but produces an *unlinked* copy — a dead end, not a contribution.
- **In-editor suggestion mode (Google-style).** Richer, but a much larger build and it presumes real-time shared editing (Phase 2). PR-based propose is the lighter, git-native first step and can coexist.
- **Do nothing / rely on view-only enforcement.** Rejected: blocking a `pull`-only user without giving them a way to contribute is the *worse* half of the permission story.

## Unresolved questions

- **Fork vs same-repo branch as the default** for a `pull`-only user — always fork, or offer branch when they have limited write?
- **PR body provenance** — auto-generate from change summaries, require the user to write it, or both?
- **Does this reduce the need for view-only enforcement further?** If contributing back is easy, “view-only” may only ever mean the hosted read view (open question #4 in [[proposals/0001-collaboration-ux|Proposal 0001]]).
- **Interaction with auto-sync** — when a user has push, does “Propose” compete with direct push? Likely an explicit mode toggle.
- **Non-GitHub remotes and the federated store** — is PR-contribution GitHub-only, or does the federated backend grow an equivalent “propose” primitive?
