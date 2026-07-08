---
description: The identity foundation under @-mentions, replies, and the notification inbox — grounded in the OK codebase and verified GitHub API docs. Why today's principal-id is per-install and you could only tag past participants; then four ways to model person-identity (GitHub delegation, git-email key, committed members file, federated accounts) with tradeoffs and a phased recommendation.
---
# Collaborator identity: who can you @-mention?

**Status:** provisional · **Owner:** shagun@inkeep.com · **Part of:** [[guides/collaboration-synthesis|Human-to-human collaboration research]] · **Hub:** [[README|Collaboration UX]] · **Feeds:** [[guides/collaboration-status|build status]] — the "notification inbox for mentions & replies" bullet

## Question

The [[guides/collaboration-status|status doc]] lists *"a notification inbox for mentions & replies"* — a per-user inbox that fires when you're @-mentioned or someone replies to your thread. That bullet, and the `@`-mention UI it implies, quietly assume something OK does not have: **a stable, enumerable notion of "a person" you can address.** Before any of it is worth building: *when you type `@`, who can you actually tag — and will the notification reach them?*

There is no single answer — there's a **design space of identity models**, each with real tradeoffs. This doc grounds the problem in the `inkeep/open-knowledge` codebase, then surveys four options.

## Three questions hiding inside "who can I `@`?"

"Mention a person" is really three sub-problems, and OK solves none of them cleanly today:

1. **Directory** — *who is even available to tag?* (the candidate list the autocomplete pulls from)
2. **Key** — *is there a stable id for a person, the same across their devices?* (so a mention resolves and a notification routes)
3. **Inbox** — *where does the notification land?* (a per-recipient, cross-document store)

**Directory and key are independent axes** — you can source them differently (key on git-email but take the directory from GitHub, or key on GitHub login but take the directory from a committed file). The options below are really combinations of those two choices. Every comparable tool answers all three via one stable, server-known person-id; see [[guides/collaboration-synthesis|the synthesis]] and the identity note in [[guides/live-blocks|Liveblocks]] (*"identity is resolved, not stored"*).

## What OK has today (codebase reality)

> First-party observations of the `inkeep/open-knowledge` code, kept separate from the GitHub-API survey. File paths are the evidence.

- **A per-project principal record — but its id is a random UUID, not a person.** *(PARTIAL — the crux)* `principal.ts` `loadPrincipal()` mints `id = ` `principal-${randomUUID()}` on first run and freezes it in `.ok/local/principal.json`. Git config supplies only the *display* fields (`display_name`, `display_email`); the id is never derived from the email. `source` is `git-config` or `synthesized`.
- **That local record is gitignored — so it is per-install, not per-person.** *(IMPLEMENTED, and it's the problem)* `.ok/.gitignore` ignores `local/`; `principal.json` is untracked. Every clone / machine mints its **own** random id. Direct evidence: the same human `shagun-singh-inkeep` is `principal-b7e49ac8…` in this repo's history but `principal-f17d598f…` in another OK repo. One person, two identities.
- **No directory of people-with-access — only participation.** *(NOT FOUND — the gap)* The only ways to enumerate people today are **live presence** (`awareness.ts` — who's connected now) and **contributor history** (`contributor-tracker.ts` — `ok-contributors:` lines in commits). Both are *participation-limited*: a teammate who has repo access but hasn't opened the doc is **un-taggable**. OK never calls the GitHub collaborators API — `github-permissions.ts` reads only the *current* user's own `permissions.push`, and `share/owners.ts` lists *your* publishable orgs, not this repo's people.
- **No account system, no shared people directory.** *(NOT FOUND)* Identity lives only (a) locally per install, (b) denormalized in git commit lines, (c) ephemerally in awareness.
- **No notification / inbox / unread anything.** *(NOT FOUND)*
- **A working `@` — pointed at pages, not people.** *(IMPLEMENTED, wrong target)* `composer-mention.ts` serializes mentions to `@path` for agent dispatch, never stores them in the doc CRDT, and lives in the "Ask AI" composer — not the document editor.

**Bottom line:** the addressable unit is a per-repo, per-machine random UUID; the only "directory" is *people who already showed up*; and there's no inbox to route to.

## Who you can tag today

| Candidate | Taggable / notifiable? | Why |
| --- | --- | --- |
| Git-configured human, **already participating** | ✅ yes | in presence now, or in contributor history |
| Agents (Claude, Cursor, Codex…) | ✅ technically | same `agent-<uuid>` taxonomy — "notify an agent" semantics differ |
| **Teammate with access who hasn't participated** | ❌ **no** | no directory of access-holders |
| **Same human on a second machine / clone** | ❌ **no** | a *different* random `principal-<uuid>`; the id doesn't follow the person |
| Synthesized user (no git name/email) | ❌ no | `source: synthesized` → `principalId` omitted from awareness |

## The design space: four ways to model "a person"

| # | Model | Key | Directory | Host-agnostic? | Lift |
| --- | --- | --- | --- | --- | --- |
| 1 | Delegate to GitHub | GitHub login | repo collaborators API | ❌ GitHub-only | low (on GitHub) |
| 2 | Git-email key | hash(git email) | git history authors | ✅ any git host | low |
| 3 | Committed members file | id in the file | `.ok/members.yml` | ✅ fully | low–med |
| 4 | Federated accounts | OK account id | workspace members | ✅ (own system) | high |

### Option 1 — Delegate to GitHub

**How it works.** The mention target is a **GitHub user**. The directory is the repo's collaborator list (`GET /repos/{owner}/{repo}/collaborators?affiliation=all`), and the key is the GitHub **login / user-id**. Because OK is git-backed and usually GitHub-hosted, GitHub already knows everyone with access and gives each a globally stable handle — so you build *neither* a directory *nor* an id scheme. Verified, `affiliation=all` returns *"outside collaborators, organization members that are direct collaborators, … access through team memberships, … default organization permissions, and organization owners"* ([[external-sources/github-list-collaborators-contributors|source]]) — everyone with access, participated or not. Same GitHub-as-source-of-truth spine as [[guides/view-only-access|view-only access]].

**The permission constraint (verified), and why it isn't fatal.** Listing collaborators requires *"write, maintain, or admin privileges"* ([[external-sources/github-list-collaborators-contributors|source]]) — a push-level token. Two reasons that's OK: (a) a mention author *is* an editor, so they hold `push` by construction ([[external-sources/github-repository-roles|repository roles]]); (b) the fetch belongs **server-side** — OK's server already holds a GitHub OAuth token for sync and can list collaborators once, serving the roster to all clients (the [[guides/live-blocks|Liveblocks]] backend-resolves-the-directory pattern). **Decision: no GitHub App** — rely on the OAuth token OK already manages for sync, not a separate App install to provision and get approved. Push-free fallback: `GET …/contributors` (no push, works on public repos) but committers-only and *"only the first 500 author email addresses … link to GitHub users"* ([[external-sources/github-list-collaborators-contributors|source]]).

**Pros.** Lowest-lift on GitHub; the full access-list for free; a key that's stable across every device and repo; reuses the view-only spine. **Cons.** Couples identity to **GitHub specifically** — breaks on GitLab / Bitbucket / self-hosted, on local-only / no-remote repos, and offline; introduces server-token custody; needs an unlinked-email → GitHub-account bridge to route to a live editing session.

### Option 2 — Git-email key

**How it works.** Use the one cross-machine-stable field already in **every commit** — the normalized git email — as the person-key: `principal-${hash(normalize(email))}`. Same email on any machine → same id, with no API and no token. The directory comes from git-history authors (widened by Option 3 if you want non-participants).

**Pros.** Host-agnostic — any git host, fully offline, zero external dependency; email is *already* the same person everywhere, so it's trivially cross-device; it's a change to how `loadPrincipal()` mints the id, nothing more. **Cons.** The directory is participation-limited (only people who've committed) unless paired with a members file; **email privacy** (a hash resists casual scraping but isn't anonymous — though the raw email is already in commit metadata); **shared or rotated emails** collapse several people into one id or split one person into two; no display name / avatar without a separate lookup.

### Option 3 — Repo-committed members file

**How it works.** A small team roster **committed into the repo** — e.g. `.ok/members.yml` listing each person's id / login / email + display name. The directory *is* that file; the key is whatever id it defines. Anyone listed is taggable even if they've never participated.

**Pros.** Works everywhere — any host, offline — with **no API, token, or permission friction at all**; it's version-controlled and reviewable (a PR adds a teammate); you can `@` the whole team, not just past participants. Nice property: it can be **generated** — snapshot GitHub collaborators (Option 1) or git authors (Option 2) into the file — turning a dynamic, permissioned lookup into a durable offline artifact. **Cons.** Someone has to maintain it (manually or via a sync job); it can **drift** from real access (a listed person who lost access, or an access-holder not yet added); it's a new convention to define and keep honest.

### Option 4 — Federated / OK-native accounts

**How it works.** Identity lives in OK's own hosted backend — workspace membership + accounts. Directory = workspace members; key = the account identity. This is the Notion / Figma model — and it is **already specced (Draft)**: the [[external-sources/federated-collaboration-sync-spec|Federated Collaboration Sync spec]] (`2026-05-20-federated-collaboration-sync`, owner Andrew) models it as `TENANT → PRINCIPAL`, each principal carrying an `inkeep_identity`.

**The database concern is softer than it first looks.** The plan is **not** a greenfield accounts table — a locked, 1-way-door decision (**D5**) is that **auth reuses `app.inkeep.com`, mapped to the OK principal model.** So Inkeep's existing auth *is* the accounts store; OK consumes a signed token and resolves it to a principal. Still open there: the token→principal mapping (Q4) and the tenancy unit — per-user vs per-org (Q3).

**Pros.** Fully general — non-git users, external guests, mobile / thin clients, avatars, invite flows — independent of any git host; the proper end-state for a hosted product, and it reuses Inkeep auth rather than building accounts from scratch. **Cons.** Gated on the whole federated backend landing (still Draft, its identity sub-questions Q3/Q4 P0-open); it duplicates what GitHub already knows for git-backed repos; and, crucially, **the spec sets the auth *direction* but not the mention *directory*** — it says nothing about "who can I `@`." This is the far side of [[guides/view-only-access|view-only]] open question #4.

### Directory in the cloud world

Everything above answers the **directory** for the *git-backed world today* — GitHub collaborators, push tokens, the contributors fallback. In the **cloud / federated world it mostly dissolves.** The [[external-sources/federated-collaboration-sync-spec|Federated Collaboration Sync spec]] models `TENANT → PRINCIPAL ("has members")`, so the directory is simply **tenant membership**: `@` anyone in your org, and the central store already has the list (it authed them via `app.inkeep.com`). No collaborators API, no push token, no 500-author cap.

Two qualifiers before banking on it:

- **"Org" has to exist first, and its shape is undecided.** The spec's **Q3 is open** — is the tenancy unit per-user or per-org? "Same org" presupposes the per-org answer. And *today* (git-only, no cloud) OK has no org of its own; "same org" would mean the *GitHub* org (`GET /orgs/{org}/members`), which is empty for a personal repo.
- **Org-wide can be too broad — phantom mentions.** In a large tenant, "anyone in the org" lets you `@` someone with no access to *this* doc, who then gets a notification they can't act on. Scope the directory to **org members ∩ has-access**, or follow Notion and prompt to **grant access on mention**.

**Net:** once the tenant model lands, "`@` anyone in the same org" is the right default directory — much simpler than the git-world machinery — modulo the per-org Q3 decision and access-scoping.

## The fork that decides it

The choice collapses onto the same question as [[guides/view-only-access|view-only]] open question #4: *is git the permanent source of truth, or does a hosted store become primary?*

- **Git stays primary** → delegate: GitHub where present (Option 1), git-email / members-file otherwise (Options 2/3).
- **Hosted store becomes primary** → federated accounts (Option 4).

Identity and permissions land on one decision — a Shagun↔Miles call, not an engineering one.

## What no option solves for free

- **The inbox is net-new regardless.** A per-recipient, cross-document store keyed by whichever person-key you pick — a server-side store (`~/.ok/local/notifications.jsonl` / SQLite, matching the existing `.ok/local/` convention) or a per-principal `Y.Doc` mirroring `__system__` agent presence. **Detection fires in a server observer** (`server-observers.ts`, `afterAllTransactions`) and must be **idempotent** — dedup on a stable `(mention-node-id, mentioned-person)` key or every edit re-notifies.
- **Synthesized / no-git users stay unaddressable** under every option (they have no email, no GitHub account, no roster entry). Gate mentionability to identified users, plus a one-time *"set a name/email so teammates can @-mention you"* nudge to promote them.

## Recommendation in one line

**Near-term (git-backed): delegate to GitHub where the repo is on GitHub (Option 1 — server-side token, no App), with a git-email key or a committed members file as the host-agnostic fallback (Options 2/3). Long-term (cloud): the directory becomes tenant membership via `app.inkeep.com` auth (Option 4).** The inbox is a net-new per-user server store filled by an idempotent server observer either way — and **git-email is the natural person-key to carry across both worlds**, so a mention authored today still resolves after the cloud store lands.

## Sequencing

The key + directory (whichever option) is a **prerequisite that sits below comments** — comments' author/mention fields need the same stable person-key, so do it before or alongside the [[guides/collaboration-status|comments "big rock"]]. The inbox follows. An inline `@person` in the document body needs none of the comment side store — so once identity resolves, a doc-body mention → inbox slice can ship independently of comments.

## Open questions

- **The person-key** — git-email is the most universal candidate; is its privacy / shared-email downside acceptable, or does a different key win?
- **Server token custody (Option 1)** — whose OAuth token lists collaborators, and what happens when it's pull-only or expired?
- **Members-file upkeep (Option 3)** — hand-maintained vs generated from GitHub/git, and how drift is reconciled.
- **Unlinked git email → account** — how often the local editor's email fails to map to a GitHub/OK account, and whether the email-hash key is enough there.
- **Cloud directory scope** — org-wide (tenant membership) vs access-scoped, plus grant-on-mention; presupposes the spec's Q3 lands on per-org tenancy.
- **The deciding fork** — git vs hosted source of truth (shared with [[guides/view-only-access|view-only]] #4).
- **Does this block comments, or precede them?** — argued as *precede*; confirm against the [[specs/human-to-human-collaboration/spec|Phase 2 spec]].

## Further reading

- Sources: [[external-sources/github-list-collaborators-contributors|GitHub list collaborators & contributors]] · [[external-sources/github-repository-roles|GitHub repository roles & permissions]] · [[external-sources/federated-collaboration-sync-spec|Federated Collaboration Sync spec (the cloud/Option 4 backend)]]
- Feeds: [[guides/collaboration-status|build status]] (the mention-inbox bullet) · [[specs/human-to-human-collaboration/spec|Phase 2 spec]] · [[proposals/0001-collaboration-ux|Proposal 0001]] (open question #4)
- Related: [[guides/collaboration-synthesis|Human-to-human collaboration synthesis]] · [[guides/view-only-access|View-only access]] (the sibling GitHub-vs-federated question) · [[guides/live-blocks|Liveblocks]] (identity resolved, not stored) · [[guides/yjs-baseline|Yjs/Hocuspocus baseline]] (awareness) · [[guides/google-docs|Google Docs]] (comments precedent)
