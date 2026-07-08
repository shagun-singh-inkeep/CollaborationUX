---
title: "View-only access: how other tools do it"
description: Survey of read-only / view-only access and permission models in Liveblocks, Google Docs, Notion, and Figma — the roles, what view-only actually restricts, whether a viewer stays visible, and where enforcement lives — with a recommendation for Open Knowledge Phase 2 (G4).
type: research
category: research
status: provisional
owner: shagun@inkeep.com
created: 2026-07-06
last_verified: 2026-07-06
sources:
  - external-sources/liveblocks-room-permissions.md
  - external-sources/google-docs-sharing-roles.md
  - external-sources/notion-sharing-permissions.md
  - external-sources/figma-share-permissions.md
tags:
  - collaboration
  - ux
  - research
  - permissions
  - view-only
  - phase-2
---
# View-only access: how other tools do it

**Status:** provisional · **Owner:** shagun@inkeep.com · **Part of:** [[guides/collaboration-synthesis|Human-to-human collaboration research]] · **Hub:** [[README|Collaboration UX]] · **Feeds:** [[specs/human-to-human-collaboration/spec|Phase 2 spec]] G4 · open question #4

## Question

Open Knowledge has no view-only state today — access is just GitHub repo access, not enforced inside the product ([[guides/collaboration-status|status]]). Phase 2's **G4** wants "a person can be given a document/project they can read but not edit, enforced by the product." Before we design it, how do the tools that already solved this — [[guides/live-blocks|Liveblocks]], [[guides/google-docs|Google Docs]], [[guides/notion|Notion]], [[guides/figma|Figma]] — actually model view-only?

## The three things "view-only" has to decide

Surveying the four tools, every view-only design is really three separate choices — and the interesting differences are in the second and third, not the first:

1. **The role ladder** — how many rungs between "can't touch it" and "owns it."
2. **What read-only actually blocks** — just typing? also commenting? also download/copy/export?
3. **Whether a viewer is still *present*** — can you see a read-only person's cursor, or do they go invisible?

## Per-tool survey

### Liveblocks — read-only, but still present

The most useful model for OK because it runs on the [[guides/yjs-baseline|same Tiptap + Yjs substrate]]. Permissions are per-room and coarse ([[external-sources/liveblocks-room-permissions|source]]):

- **`room:write`** — view and edit.
- **`room:read` + `room:presence:write`** — read the document's storage, **but still edit your own presence.** So a read-only client can *broadcast its cursor* while being unable to change a single character.
- Set at three levels — `defaultAccesses` (whole room), `groupsAccesses` (a whole **team/workspace** granted `*:read` or `*:write` in one entry), `usersAccesses` (a per-user allow); precedence is user › group › default ([[external-sources/liveblocks-room-permissions|source]]).
- Enforced server-side: with **ID tokens** (recommended) permissions live in the backend and are "checked at the door" each time you enter; the room is the source of truth.

**The load-bearing idea:** *"can edit the document" and "can broadcast presence" are separate permissions.* A viewer can be visible without being able to write — exactly the nuance flagged in the [[guides/collaboration-synthesis|synthesis]].

### Google Docs — a role ladder, and read-only means more than "can't type"

Four rungs ([[external-sources/google-docs-sharing-roles|source]]):

- **Viewer** — read + download (by default); no comment, edit, or share.
- **Commenter** — view + comment + download; no edit.
- **Editor** — view + comment + edit + share; not ownership.
- **Owner** — everything.

Two details worth stealing:

- **Commenting is its own rung.** "Read but let me comment" is a first-class, common state — not folded into edit.
- **Download/print/copy is a *separate* restriction.** Owners can turn off "viewers and commenters can download, print, and copy" — off is the safer default. View-only isn't only about the keyboard; it's also about exfiltration. There's also a distinct **publish-to-web** read view (a hosted, no-account read surface).

### Notion — the strictest view-only, plus inheritance

Four levels ([[external-sources/notion-sharing-permissions|source]]):

- **Can view** — read only; **cannot even comment**, edit, or share. (Stricter than Google's Viewer.)
- **Can comment** — view + comment.
- **Can edit** — edit but not share.
- **Full access** — edit + share.

Two details:

- **Public web links** let someone view without a Notion account; commenting/editing still needs login — a clean "anyone with the link can read" tier.
- **Broadest-access wins.** If a user already has Full access via a parent page or teamspace, a narrower per-page setting won't *reduce* it. Permissions **inherit down a tree** and the widest grant applies — directly relevant to OK, where docs live in a folder/repo hierarchy.

### Figma — binary view/edit, viewer stays live

The simplest ladder ([[external-sources/figma-share-permissions|source]]): **can view** or **can edit**, nothing between.

- A **viewer is still a live participant** — they can inspect and they still show up in the file. Presence is decoupled from edit rights, same as Liveblocks (see the [[guides/figma|Figma study]]).
- **Copy/share/export** is again a separate *advanced* toggle ("allow viewers to copy, share, and export"), off by default — the same exfiltration boundary Google draws.
- **Prototype-only links** present a design to stakeholders without granting access to the source file — a purpose-built, narrower-than-view read surface. (Caveat: on Starter, presentation viewers can still reach the source file — a reminder that a read view has to be *actually* sealed, not just a different URL.)

## Cross-cutting patterns

| Question | Liveblocks | Google Docs | Notion | Figma |
| --- | --- | --- | --- | --- |
| Role ladder | write / read (+presence) | Viewer · Commenter · Editor · Owner | View · Comment · Edit · Full | View · Edit |
| Viewer can comment? | n/a (infra) | ❌ (Commenter can) | ❌ (Can comment can) | via comment tier |
| Viewer still visible (presence)? | ✅ read + presence:write | caret + name flag | avatar/cursor | ✅ stays live |
| Download/copy/export | — | separate toggle, **off** default | not specified | separate toggle, **off** default |
| Hosted no-account read view | room link | publish-to-web | anyone-with-link | link / prototype-only |
| Enforcement | server, "checked at the door" | server-side role | server-side, inherits down tree | server-side |

Four patterns fall out:

1. **Presence is a *separate* permission from editing.** Both Liveblocks and Figma let a read-only person stay visible. View-only should mean "can't change the doc," not "disappears."
2. **"Read-only" is a spectrum, not a bit.** Can they comment? download? copy? export? The mature tools split these apart, and default the exfiltration ones (download/copy/export) **off**.
3. **Enforcement is server-side and non-negotiable.** Everyone checks at the door — the read-only client literally cannot mutate state, it's not just a hidden button. Hiding edit UI is not the same as refusing writes.
4. **A hosted, no-account read link is table stakes.** Every tool has a "send someone a link and they can read it without an account" surface, separate from inviting a collaborator.

## What OK has today (codebase reality)

> First-party observations of the `inkeep/open-knowledge` code, kept separate from the third-party survey above. File paths are the evidence.

OK has **no view-only state** — but the survey's building blocks are unevenly present:

- **No read-only editor render.** *(NOT FOUND)* The editor never goes non-editable — `isEditable` is *read* in a few UI guards but `setEditable(false)` is never called, and there's no `readOnly`/`viewer` prop on `TiptapEditor.tsx` or `SourceEditor.tsx` (no CodeMirror `EditorState.readOnly`).
- **No write-path authorization — the crux.** *(NOT FOUND)* Document mutations flow over Hocuspocus with **no per-client permission check** (no `beforeHandleMessage` / `onLoadDocument` gate); any connected peer can publish edits. Enforcement today is *implicit and post-hoc*: writes land in the Y.Doc freely and git-commit authorship records who did it (`api-extension.ts`). So OK currently fails the spec's "refused at every write surface" test **by construction — there is no surface that refuses.**
- **No roles.** *(NOT FOUND)* No viewer/editor/`canEdit` concept anywhere; the only permission notion is the GitHub push check.
- **The GitHub push gate is about sync, not editing.** *(PARTIAL — wrong layer for G4)* `github-permissions.ts` checks `permissions.push` from the GitHub API and, if false, pauses *git sync* with a toast (`use-no-push-permission-toast.ts`); the editor stays fully editable. It's an orthogonal sync concern, not a view-only mechanism.
- **Presence is already decoupled from edit rights.** *(IMPLEMENTED)* Awareness publishes cursor/name/color/mode unconditionally, independent of any write capability (`TiptapEditor.tsx` `setLocalState`, `awareness-user.ts`, `use-presence.ts`). So the Liveblocks/Figma "a viewer can still broadcast presence" pattern is **already true in OK, for free** — we don't have to build it.
- **`share_link` is a pull-into-your-own-install model, not a hosted read view.** *(IMPLEMENTED, pull-model)* `share-url.ts` encodes a GitHub blob URL into a share link and `publish.ts` is a share-to-GitHub wizard — the recipient clones into their *own* OK instance; there's no central hosted read surface.

**Bottom line:** the two real gaps for G4 are a **read-only editor render** and — more importantly — an **actual authorization gate on the write path**, which does not exist at all today. Presence-decoupling is already done; `share_link` is the pull-model starting point a hosted read view would bridge from.

## What this means for Open Knowledge (G4)

Tied to the [[specs/human-to-human-collaboration/spec|Phase 2 spec]] and proposal [[proposals/0001-collaboration-ux|open question #4]]:

- **Start binary (Figma-style): viewer / editor.** The spec already calls for a minimal viewer/editor split; the survey backs that — Figma ships on two rungs. Add a **commenter** rung only when comments land (Google/Notion treat it as first-class, but it's meaningless before OK has comments).
- **Enforce by refusing every write surface, not by hiding UI — this is the core build.** The spec's test — "a view-only participant cannot mutate via *any* write surface (editor, paste, drag, API)" — is exactly the server-side, checked-at-the-door model all four tools use. Today OK has **neither** a read-only render **nor** any write-path check (see *What OK has today*): the Hocuspocus write path authorizes no one, so a viewer can't be refused because *nothing* is refused. The real work is adding an authorization gate at the write layer; a read-only editor render is necessary but, on its own, just hides buttons.
- **Keep viewers present — already done.** The Liveblocks/Figma decoupling (a viewer broadcasts a cursor but can't write) is **already true in OK's code**: awareness publishes presence unconditionally, independent of write capability (`awareness-user.ts`, `use-presence.ts`). So once a write-path gate exists, a view-only participant will *still show a cursor* for free — no extra work. This is the one piece of the survey OK already nailed.
- **Decide the enforcement point (the real open question).** Google/Notion/Liveblocks all own roles in a server. OK's #4 is whether the **federated store owns roles** or we stay **git-delegated with a thin UI veneer**. The survey's lesson: whoever owns it must enforce at the write path, because a git-delegated veneer that only hides buttons fails the "any write surface" test. Notion's **inherit-down-the-tree, broadest-grant-wins** model is the closest analog for OK's folder/repo hierarchy and worth copying if the federated store owns roles.
- **Treat a hosted read view as its own workstream.** "Does view-only imply a hosted read view (a shareable link, no OK install)?" — the spec's open question — maps onto publish-to-web / anyone-with-link / prototype-only. It's a *bridge from today's* `share_link` — which in code (`share-url.ts`, `publish.ts`) is a **pull-into-your-own-install** model: the link encodes a GitHub blob URL and the recipient clones into their own OK instance. A hosted read view is the missing surface between that and "send a link, they just read it." Figma's Starter leak is the warning: a read view has to be genuinely sealed server-side, not just a different URL over the same file.
- **Defer download/copy/export restrictions.** Real, but a later hardening — not v1. Worth noting OK's twist: the content is local-first markdown in a git repo, so "prevent copy" is far weaker a guarantee than in a hosted tool. Be honest about what view-only can promise on a local substrate.

## Do we even need to build permissioning?

**Mostly no.** For a git-backed, local-first tool the honest answer is: GitHub already provides the only hard permission boundary that exists, and a local clone can't truly be locked anyway. The trap is treating "permissioning" as one thing — it's really two different asks:

- **Enforcing who can change the canonical doc** — GitHub already does this, completely. `push` vs `pull`: a read-only collaborator clones, edits a fork, can't push, and contributes via PR *on GitHub* (OK itself has no in-product PR flow — see the reality check below). That's a full view/edit model OK builds **nothing** for. Private repos (no access → no clone), public repos (free read tier), and team/workspace grants (GitHub teams) all come with it.
- **A view-only *experience*** — read-only render, a legible fork state, a hosted read link. That's UX and *sharing*, not a permission system.

**The one real gap git can't cover:** real-time shared live sessions over the federated server — there git isn't in the loop, so a connected peer can broadcast edits with nothing to stop them. That's the only place a native write-path gate earns its keep, and it only exists once the federated backend ([[specs/human-to-human-collaboration/spec|Phase 2]] dependency, still Draft) ships.

**Worth doing, and small (UX, not permissioning):**

- **Make the fork state legible** — "you're editing your own copy; it won't sync to the original" — off the `permissions.push` signal OK already fetches. Highest value, and it isn't a permission system.
- **Add a “propose changes / open PR” action** — the genuinely missing primitive (verified: OK creates repos but never opens PRs). This is what turns the copy from a Google-style dead end into a real fork-and-contribute loop, and it's plausibly worth more than any view-only enforcement. Specced as [[proposals/0002-contribute-from-ok|Proposal 0002]].
- **A hosted, sealed read view** (show a doc to a non-collaborator, no repo access) — a *sharing* feature; scope it on its own when a concrete need appears.

**The honest limit:** on a local clone the bytes are on someone's disk — "view-only" is UI courtesy, not a guarantee. Real read-only exists only on a server-mediated surface.

**So G4 is over-specified.** "View-only enforced by the product" should be reframed to: *make GitHub's existing permission legible, and enforce natively only in live federated sessions.* No roles table, no RBAC, no per-doc ACL.

**The one thing that would flip this:** if OK's source of truth shifts from git to a hosted/federated store with users who aren't GitHub collaborators, GitHub drops out of the loop and you *would* need real OK-native roles. So the decision that actually gates this isn't "build permissioning y/n" — it's **"is git the permanent source of truth, or does the hosted store become primary?"** — a Shagun↔Miles call tied to [[proposals/0001-collaboration-ux|open question #4]].

## Recommendation: view/edit permissions with GitHub

*(How to do the minimum above — if and when you build the enforcement, lean entirely on GitHub rather than a new role system.)*


The question behind open question #4, made concrete: OK is git-backed, and **GitHub already knows who can read vs write a repo.** So the recommendation is *don't invent a parallel role system* — **let GitHub own the roles (git-delegated), and enforce its answer in two layers.**

### 1. GitHub is the role source of truth

Map OK's viewer/editor split straight onto the GitHub `permissions` object OK already fetches ([[external-sources/github-repository-roles|source]]; OK reads it in `github-permissions.ts`):

| GitHub access (authenticated user's `permissions`) | OK role |
| --- | --- |
| `push` / `maintain` / `admin` = true | **Editor** |
| `pull` = true, `push` = false (incl. **Triage**) | **Viewer** — read, or fork-and-edit (contribute-back = manual GitHub PR; not in OK yet) |
| public repo, anonymous / not a collaborator | **Viewer** (read-only) |
| private repo, no access | **No access** (no clone → no doc) |

Why delegate rather than build a role store:

- **It's already the system users manage.** Teams already grant repo access on GitHub; a second permission UI in OK would drift out of sync. GitHub roles even apply uniformly to members, outside collaborators, and teams — so "a whole team can view" comes for free (the workspace-grant idea Liveblocks does with `groupsAccesses`, but native to git).
- **The signal is already wired.** `github-permissions.ts` calls `GET /repos/{owner}/{repo}` and reads `permissions.push`. Today OK only uses it to pause *sync*; the same call answers "viewer or editor?" with no new backend.
- **Least surprise:** "if you can push on GitHub, you can edit in OK; if you can only read, you're a viewer."

### 2. "Viewer" in a git world = *edit your own copy, can't push to the original*

Here's the twist the local-first + git model adds — and it's exactly the Google Docs **Make a copy** analogy: a `pull`-only user isn't "can't touch it," they're **"can edit their own clone, but can't push to the canonical doc."** That's the git fork model — the same shape as Google's *Make a copy*. Git *can* have a **way back** a Google copy lacks (a pull request) — but, reality-check, **OK doesn't open PRs today** (see the callout below), so that advantage is latent, not shipped.

Because of that, "read but not write" usefully splits in two:

- **Fork-and-edit (the copy).** `pull` access → edit a personal clone freely; changes never reach the canonical doc. OK's `share_link` / publish is a *partial* version of this — it creates a **new repo** with your copy (`share-url.ts`, `publish.ts`) and the recipient pulls into their own install. But that's a plain *copy*, not a linked fork: **there is no PR back to the original.** So today OK's copy is a **Google-style dead end**, not yet the git fork-with-a-way-back.
- **Pure read (the hosted view).** For someone who should *only* look — a public-repo reader or a hosted read view — render it genuinely read-only (the Google "Viewer, can't copy" end).

> **Reality check — OK cannot open a PR today (verified in code).** The only GitHub write OK performs is `repos.createForAuthenticatedUser` / `createInOrg` in the share/publish flow (`publish.ts`) — it creates a **new repo** with your copy and pushes there. There is **no `pulls.create` / open-a-PR path anywhere** in the product. So right now a `pull`-only user edits locally, the commits pile up, and sync simply **pauses** (`github-permissions.ts` → a toast) — there is *no in-product way back to the original*. Contributing back means the user leaving OK and opening a PR on GitHub by hand. **The git "way back" only becomes real if OK adds a propose/PR action** — arguably higher-value than any view-only enforcement.

So "just force the editor read-only" is too blunt. Enforce by **context**, using the same `push` bit:

- **Async / local clone — don't lock the keyboard.** Forcing read-only here discards the make-a-copy capability git gives for free. Instead make it **legible**: "you're editing your own copy; these changes won't reach the original" (and OK can't open a PR for you today — that's the primitive still to build). Today OK silently lets a `pull`-only user edit and only *then* toasts a paused push — the fix is to *name it a fork*, not to disable typing.
- **Real-time shared canonical session — here read-vs-write bites.** A `pull`-only peer must not mutate the *shared* canonical doc, and OK has **no write-path authorization** today (see *What OK has today*). Two clean options, the same `push` bit deciding: the **federated server refuses writes** at the Hocuspocus/write layer (true read-only), **or** it **branches the viewer into their own live editable fork**.

**This answers open question #4:** GitHub owns *who* (roles); enforcement is **contextual** — a legible fork on a local clone, write-path refusal (or auto-fork) in a shared session — **not** a parallel role store. And it *could* turn OK's git substrate into a feature: view access carrying "… and you can fork and propose back" would be strictly more than Google Docs offers — but only **once OK ships a PR/propose flow.** Until then the fork is a dead-end copy (see reality check).

### 3. Handle the edges

- **Public repos give you the free read tier.** Anyone can `pull` → anonymous = viewer. That's OK's "anyone with the link can read" surface (the Google publish-to-web / Notion anyone-with-link tier) with no new mechanism — and the natural gate for a future hosted read view.
- **Keep viewers present.** A GitHub-read-only user still broadcasts a cursor — already true in OK (presence is decoupled; see above). GitHub gates *writes*, never *presence*.
- **Re-check on change.** Cache `permissions` with a TTL and refresh on reconnect; a revoked `push` should drop the user to viewer mid-session. OK already caches per session — extend it to re-poll.
- **Be honest about local-first limits.** On a purely local/offline clone, "view-only" is a UI courtesy, not a hard guarantee — the hard guarantee exists only where the federated server mediates writes. Don't promise more than the substrate can enforce.

### In one line

**GitHub `push` = edit the original; `pull`-only = edit your *own copy* (contribute-back via PR isn't in OK yet — a primitive worth building). Make that a legible fork on a local clone, and have the federated server enforce the same `push` bit at the write path (or auto-fork) for live shared sessions — no parallel role store.**

## Open questions

- **Enforcement point** — federated store roles vs git-delegated veneer (proposal #4). Everything above assumes writes are refused server-side; a git-only model may not be able to make that promise.
- **Local-first exfiltration** — if the repo is on someone's disk, can "view-only" mean anything beyond UI, or only the hosted read view is truly sealed?
- **Does view-only need its own identity/invite flow** — link vs account vs GitHub identity (ties to the federated backend's auth, and to the [[specs/human-to-human-collaboration/spec|Phase 2 spec]] invite question).
- **Presence for viewers** — do we *want* viewers visible by default (Liveblocks/Figma) or is a silent lurker sometimes the right call?

## Further reading

- Sources: [[external-sources/liveblocks-room-permissions|Liveblocks room permissions]] · [[external-sources/google-docs-sharing-roles|Google Docs sharing roles]] · [[external-sources/notion-sharing-permissions|Notion sharing & permissions]] · [[external-sources/figma-share-permissions|Figma share & permissions]]
- Product studies: [[guides/live-blocks|Liveblocks]] · [[guides/figma|Figma]] · [[guides/google-docs|Google Docs]] · [[guides/notion|Notion]] · [[guides/yjs-baseline|Yjs/Hocuspocus baseline]]
- Feeds: [[specs/human-to-human-collaboration/spec|Phase 2 spec]] (G4) · [[proposals/0001-collaboration-ux|Proposal 0001]] (open question #4) · [[guides/collaboration-synthesis|synthesis]] · [[guides/collaboration-status|build status]]