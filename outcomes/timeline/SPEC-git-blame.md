# Highlight-to-blame — Spec

**Status:** Draft
**Owner(s):** Shagun Singh
**Last updated:** 2026-07-10
**Baseline commit:** `078000202`
**Links:**

- Sibling spec (author recovery, reused here): `[specs/2026-07-06-timeline-upstream-author-attribution/SPEC.md](../2026-07-06-timeline-upstream-author-attribution/SPEC.md)`
- Shadow-repo git handle + commit model: `[packages/server/src/shadow-repo.ts](../../public/open-knowledge/packages/server/src/shadow-repo.ts)`
- Per-doc history assembly (blame-source analysis): `[packages/server/src/timeline-query.ts](../../public/open-knowledge/packages/server/src/timeline-query.ts)`
- History/diff API schemas (endpoint pattern to mirror): `[packages/core/src/schemas/api/history.ts](../../public/open-knowledge/packages/core/src/schemas/api/history.ts)`
- Contributor identity trailer (`ok-contributors:`): `[packages/server/src/contributor-tracker.ts](../../public/open-knowledge/packages/server/src/contributor-tracker.ts)`
- Editor substrate (`Y.Text('source')` holds file bytes): `[public/open-knowledge/AGENTS.md](../../public/open-knowledge/AGENTS.md)` §Editor substrate
- Timeline UI (attribution display precedent): `[packages/app/src/components/TimelinePanel.tsx](../../public/open-knowledge/packages/app/src/components/TimelinePanel.tsx)`

---

## 1) Problem statement

- **Who is affected:** Anyone reading or collaborating on a shared OK document who sees a specific passage and wants to know **who wrote this piece of text, when, and (eventually) why**.
- **What pain / job-to-be-done:** Attribution in OK today is **document-level**. The Timeline panel answers "who changed this doc, when" (per doc-level shadow WIP commit), but it cannot answer "who wrote *this sentence*." To attribute a specific passage a user must leave the app, open the project's git history, and reconstruct blame by eye — and even then the `Auto-save:` commit model attributes lines to whoever ran the sync, not the real author.
- **Why now:** Collaboration surfaces are the current focus (`[outcomes/ok-collaboration-experience/OUTCOMES.md](../../outcomes/ok-collaboration-experience/OUTCOMES.md)`). "Who wrote this?" is the highest-frequency provenance question and the natural companion to the Timeline. The author-recovery machinery being built for the sibling attribution spec makes a *correct* blame answer newly cheap to assemble.
- **Current workaround(s):** Open GitHub / `git log` / `git blame` in a separate tool and mentally map lines back to the passage; accept that authorship is often the sync committer.

### Root cause (of the gap, verified)

- Attribution is accumulated at doc granularity: `recordContributor(docName, writerId, …)` keys by `writerId` with a `Set<docName>` — **no text offsets or ranges** (`contributor-tracker.ts`). There is no per-character or per-range provenance persisted anywhere the UI can query.
- The editor has no selection→author lookup wired. A grep of `packages/app/src/editor` for `blame`/`clientID`/`provenance` finds only CodeMirror's line-number gutter and the Yjs provider client id — no blame surface.

## 2) Goals

- **G1:** Let a user select a passage in the OK editor and see **who authored it, when, and in which commit** — line-granular, in a lightweight in-app popover, without leaving the app.
- **G2:** Show the **real content author**, not the `Auto-save` sync committer — by reusing the author-recovery already specified in the sibling attribution spec (parse the `ok-contributors:` trailer / recover `%an`/`%ae`).
- **G3:** Reuse existing primitives — the shadow git handle (`shadowGit`), the history/diff HTTP + Zod schema pattern (`history.ts`), the `Y.Text('source')` file-bytes substrate for range mapping, and the Timeline's contributor display/color-seed conventions — rather than inventing new attribution machinery.
- **G4:** Degrade honestly. Where authorship cannot be recovered at real fidelity (uncommitted working-copy edits, unreadable range, blame miss), say so explicitly rather than guessing an author.

## 3) Non-goals

- **NG1:** **Per-character / pre-commit CRDT provenance** (threading stable writer identity into the Yjs `clientID` and resolving a selection's origin authors live). Highest fidelity but a substantial new subsystem — this is the deferred "Strategy B" direction from the sibling spec. Deferred (§17).
- **NG2:** **Full inline blame gutter** (a persistent per-line author strip alongside the whole document, à la editor blame annotations). v1 is on-demand — either selection-scoped (UX-A) or author-filtered (UX-B). An always-on all-authors tint is deferred (§17).
- **NG5:** **Full-history authorship** ("every line this author ever wrote, including lines later overwritten"). Both UX variants use `git blame` = **current line ownership** (last writer per line). Historical/overwritten authorship is Strategy-B territory (NG1). Deferred (§17).
- **NG3:** **Rewriting history or backfilling** authorship on commits that already landed with sync-committer identity beyond what trailer/`%an` recovery yields. Forward-and-best-effort only.
- **NG4:** **Blaming non-`.md`/`.mdx`** content, `__system__`/`__config__` synthetic docs, or content outside `contentRoot`.

## 4) Personas / consumers

- **P1 — Reader/reviewer on a shared repo:** Reads a passage, wants attribution in one gesture (select → popover). Primary consumer.
- **P2 — Collaborator resolving intent:** Uses "who + when + which commit" as the entry point to open that commit's context (Timeline / diff) for the "why."
- **P3 — Blame HTTP endpoint:** A new `GET /api/blame` consumed by the editor; mirrors the existing history/diff read-endpoint contract.
- **P4 — "Who worked on this file?" reader (UX-B):** Opens a per-file Authors panel to see everyone who has edited the doc, and clicks an author to highlight the lines that author currently owns. Inverse gesture to P1; same blame engine.

## 5) User journeys

1. Ana reads `architecture.md`, selects a paragraph, and invokes "Blame selection." **After:** a popover shows *"Ben — 3 days ago — `imported a1b2c3d`"* (Ben being the recovered content author, not the sync committer), with Ben's email-seeded avatar color. From the popover she can jump to that commit in the Timeline.
2. Ben selects a passage he just typed that has **not yet been committed/pulled** into the project repo. **After:** the popover says *"Working copy — not yet in history"* (no fabricated author). No git identity is invented.
3. A user selects a range spanning **two authors' lines**. **After (v1 default):** the popover lists the distinct authors in the range (most-recent first) with per-author line counts, or names the dominant author with a "+N others" affordance (Q1).
4. **(UX-B)** Ana opens `architecture.md` and expands the **Authors** panel. **After:** it lists everyone who has edited the doc (from existing doc-level contributor data — no new mapping). She clicks "Ben" and, **in source mode**, every line Ben currently owns is tinted with his avatar color; clicking again clears it. In WYSIWYG mode the panel still lists authors, but per-line highlight is disabled/blocked (same mapping wall — Q6/D6).

## 6) Requirements

### Functional

| Priority | Requirement | Acceptance criteria |
| -------- | ----------- | ------------------- |
| P0 | FR1 — Selection → line range (source mode) | In **source mode**, a helper maps the CodeMirror selection to a 1-based inclusive on-disk line range via the `Y.Text('source')` view. **Exact and trivial**: CM is bound directly to `Y.Text('source')` (`SourceEditor.tsx`), so selection offsets are source offsets and `doc.lineAt(pos)` yields the line (the outline-nav code already does this). Collapsed selection → caret line. |
| P1 | FR1b — WYSIWYG selection mapping | In **WYSIWYG mode** there is **no** ProseMirror-position → source-offset map (verified — the only primitive, `serializeWysiwygSelection`, re-serializes the slice to markdown *text*, not offsets). v1 either (a) gates blame to source mode (D6) or (b) maps the selection's top-level **block(s)** to source via parse-time position attrs for block-granular blame — its own spike (Q6/§15b). Sentence-precise WYSIWYG blame is out of scope for v1. |
| P0 | FR2 — Blame a line range | A server helper runs `git blame -L <start>,<end> --line-porcelain -- <path>` against the **project repo** for the doc's file and returns per-line `{ sha, author, authorEmail, timestamp }`. |
| P0 | FR3 — Recover real author | Each blamed commit's identity is resolved through the sibling spec's author recovery (parse `ok-contributors:` trailer, else `%an`/`%ae`) so `Auto-save:` sync commits surface the **content author**, not the sync committer. |
| P0 | FR4 — Honest degradation | Uncommitted lines (git blame `^` boundary / not-yet-committed), unreadable ranges, non-ancestor/detached states, and non-`.md`/`.mdx` or out-of-`contentRoot` docs return an explicit `unattributed` state — never a guessed identity. |
| P0 | FR5 — Blame endpoint | `GET /api/blame?docName=…&startLine=…&endLine=…&branch=…` returns a Zod-validated response (schema colocated with `history.ts`), collapsing per-line results into distinct author spans; loopback/same-origin only, matching the existing read-endpoint posture. |
| P1 | FR6 — In-editor UI | Selecting text exposes a "Blame selection" affordance (context action / command) that opens a popover rendering author name, relative time, short SHA, and email-seeded avatar color, reusing Timeline display conventions. Multi-author ranges per Q1. |
| P1 | FR7 — Jump to commit | The popover links to the blamed commit's Timeline entry (or diff) so the user can move from "who/when" to "why." |
| P1 | FR8 — Per-file author list (UX-B) | A per-file "Authors" panel lists everyone who has edited the doc. **Reuses existing doc-level contributor data** (Timeline `contributors[]`); needs **no** line-mapping and works in both modes. Each entry shows name + email-seeded avatar color. |
| P1 | FR9 — Author-filter highlight (UX-B, source mode) | Clicking an author in the panel tints the lines that author **currently owns** in source mode. Derived by grouping the file's `git blame` output (FR2/FR3) by recovered author into line ranges, then decorating those CM line ranges (CM highlights line ranges natively). "Currently owns" = last-writer per line (not full history — see NG5). WYSIWYG per-line highlight is disabled (D6/Q6). Toggle off clears the tint. |

### Non-functional

| Priority | Requirement | Acceptance criteria |
| -------- | ----------- | ------------------- |
| P0 | NFR1 — No guessed identity | Never fabricate an author. Below-real fidelity (uncommitted, unresolvable) renders as `unattributed`, not a default name. |
| P0 | NFR2 — Cardinality discipline | No raw paths / free-form user strings / author identities on span or metric attributes (STOP rule). Blame spans use `doc.name` + bounded counters only; identities stay in the response body. |
| P0 | NFR3 — Path safety | `docName` → path resolution reuses the existing `contentRoot` + path→docName helper and realpath-escape guard; reject out-of-`contentRoot` and synthetic docs (`isSystemDoc`/`isConfigDoc`). |
| P1 | NFR4 — Bounded cost | One `git blame -L` per request, bounded by the selected range; failures degrade to `unattributed` and never block the editor. Optional short-TTL cache keyed by `(docName, sha-of-file, range)`. |

## 7) Success metrics & instrumentation

- A user can select any committed passage and see the recovered content author + commit in the popover (manual + integration assertion).
- Uncommitted selections render `unattributed` with zero fabricated identities (integration assertion on the working-copy path).
- Optional bounded counter: `blame` requests split by `attributed` vs `unattributed` outcome (no identities as labels; NFR2).

## 8) Current state (how it works today)

- **Doc-level attribution only.** `recordContributor` (contributor-tracker.ts) accumulates `writerId → { displayName, colorSeed, docs: Set, … }`. No offsets. The Timeline (`GET /api/history`) renders these as per-doc rows (`HistoryEntry.contributors[]`, `history.ts`).
- **Shadow repo is NOT a line-coherent blame source (verified).** Shadow WIP commits live on **per-writer refs** `refs/wip/<branch>/<writerId>` (`commitWipInner`, shadow-repo.ts ~L390). Each ref seeds its index from *its own* prior tree (`${ref}^{tree}`) then stages the **full** `contentRoot`. A `git blame` against any single writer ref therefore attributes **every** line to that writer's commits, not the real per-line author. `timeline-query.ts` reconstructs the timeline by walking *multiple* refs and merging by timestamp — there is no single linear per-doc ref to blame coherently. → Shadow-repo blame is rejected as the source (§9 Alternatives).
- **Project repo IS line-coherent** but attributes lines to `Auto-save:` sync commits (the sync committer, not the author) — the exact identity-loss the sibling attribution spec addresses; its `ok-contributors:` trailer + `%an` recovery is the fix we compose (FR3).
- **Editor substrate exposes file bytes.** `Y.Text('source')` holds the full doc source (YAML frontmatter region + body) and is what CodeMirror binds to (AGENTS.md §Editor substrate). This is the substrate FR1 maps a selection through to on-disk lines.
- **Read-endpoint pattern exists.** `history.ts` defines GET read schemas (`HistorySuccessSchema`, `HistoryVersionSuccessSchema`, `WorkspaceSuccessSchema`) with `StandardSchemaV1` + `.loose()`; `GET /api/history`, `/api/history/<sha>`, `/api/diff`, `/api/workspace` are the shape to mirror for `/api/blame`.

## 9) Proposed solution (Route 1 — project-repo line blame + author recovery)

Blame the on-disk `.md`/`.mdx` in the **project repo** for the selection's line range, then recover the real content author of each blamed commit using the sibling spec's machinery. Line-coherent (git does the hard part), and correct on identity (trailer/`%an` recovery), at the cost of line (not character) granularity and committed-only coverage.

### System design (vertical slice)

**Step 1 — `selectionToLineRange(docName, selection)` (client, `packages/app`).** Two modes, materially different:

- **Source mode (exact, cheap):** CodeMirror binds directly to `Y.Text('source')` (`EditorState.create({ doc: ytext.toString() })` + `yCollab(ytext, …)`, `SourceEditor.tsx`). The selection is already a source offset; `doc.lineAt(from)`/`doc.lineAt(to)` give the 1-based line range directly (the outline-nav path already uses `doc.line(n)`). Frontmatter needs no special handling — `source` includes the FM region and the byte-mapping is verified 1:1 (§12). Collapsed selection → caret line.
- **WYSIWYG mode (no offset today):** `Y.XmlFragment` and `Y.Text('source')` are **separate CRDTs** reconciled by re-deriving observers, not by position mapping — there is **no** maintained PM-position → source-offset map. The only selection primitive, `serializeWysiwygSelection` (`edit-with-ai-selection.ts`), re-serializes the slice to markdown *text*. Locating that text in the source by substring search is ambiguous (duplicate passages) and defeated by source-form↔canonical normalization (`_foo_`↔`*foo*`, table padding). So WYSIWYG is **not** sentence-precise-mappable with today's primitives.

**v1 decision (D6):** ship **source-mode-only** blame. WYSIWYG blame is either deferred (§17) or a separate **block-granular** design — map the selection's covering top-level block node(s) to a source line range via parse-time position attrs (`position-slice` / `sourceRaw`), accepting block granularity and post-edit staleness — which requires its own spike (Q6/§15b).

**Persistence debounce (both modes):** if the range covers bytes not yet flushed to disk / committed, mark those lines `working-copy` locally so the UI can short-circuit without a round-trip (defense-in-depth with FR4).

**Step 2 — `blameRange(shadow/project git, contentRoot, docName, start, end, branch)` (server, near the history/diff helpers).** Resolve `docName → path` via the existing `contentRoot` + path→docName helper with the realpath-escape guard (NFR3). Run `git blame -L <start>,<end> --line-porcelain --no-color -- <path>` against the **project** working repo. Parse porcelain into per-line `{ sha, author, authorEmail, authorTime, boundary }`. On any git error / non-repo / boundary-only range → return empty (→ `unattributed`, FR4/NFR1).

**Step 3 — author recovery (reuse sibling spec).** For each distinct blamed `sha`, resolve the real content author: read the commit's `ok-contributors:` trailer when present (same parser the Timeline uses), else fall back to `%an`/`%ae`. Merge-commit authorship follows the sibling spec's `--no-merges` / last-non-merge-author rule (D2 there). This is the single source of "who really wrote it."

**Step 4 — collapse to author spans + endpoint (FR5).** Collapse per-line results into contiguous distinct-author spans; compute per-author line counts for the range. Expose `GET /api/blame?docName&startLine&endLine&branch` returning a Zod-validated `BlameSuccessSchema` (colocated with `history.ts`): `{ spans: [{ author, authorEmail, colorSeed, sha, timestamp, lineStart, lineEnd }], unattributedLines: number, dominantAuthor? }`. Loopback/same-origin posture identical to the other read endpoints.

**Step 5 — UI (FR6/FR7).** On a non-empty selection, expose a "Blame selection" command/context action. It calls `selectionToLineRange` → `GET /api/blame` → renders a popover: author name + email-seeded avatar color (Timeline conventions), relative timestamp, short SHA, and a link that opens that commit in the Timeline/diff (FR7). Multi-author ranges render per Q1. Any new user-facing string goes through Lingui. Use shadcn primitives (`Popover`, `Button`) per subtree rule.

**Step 6 — degradation surface.** `unattributed`/`working-copy` ranges render an explicit "not yet in history / working copy" state in the popover — never a name (NFR1).

### UX-B — author-filter blame overlay (alternative / companion v1 shape)

Same engine, inverse gesture. Instead of "select → who wrote this," it's "pick an author → highlight what they own."

- **Author list (FR8) — free.** Reuse existing **doc-level** contributor data (Timeline `contributors[]`, already surfaced by `GET /api/history`). No line mapping, works in both modes. This piece ships independent of everything below.
- **Author-filter highlight (FR9) — source mode.** Blame the **whole file** once (`git blame` + author recovery, Steps 2–3 over the full doc rather than a selected range), group lines by recovered author into ranges, and on author-click decorate those CM line ranges with the author's color. Endpoint: either extend `GET /api/blame` to accept a whole-file (no `startLine`/`endLine`) request returning all spans, or add `GET /api/blame?docName=…` full-doc mode — same schema (`spans[]`). WYSIWYG per-line highlight disabled (D6/Q6); the author list still shows.
- **Why it's attractive:** avoids the awkward "invoke blame on a selection" gesture; the author-list half is free and useful everywhere; the highlight half is pure source-mode line decoration (no rendered-tree mapping). Same `git blame` + recovery backend as UX-A. **Open: is this the primary v1, a companion, or a fast-follow? (Q7.)**

### Alternatives considered

- **Shadow-repo blame (rejected — verified).** Attractive because shadow commits already carry writer identity, but per-writer refs each accumulate the full content tree, so single-ref blame mis-attributes every line to the ref's writer, and there is no single linear per-doc ref (timeline-query merges many refs by timestamp). Not line-coherent. See §8.
- **CRDT per-range provenance (Strategy B — deferred, NG1/§16).** Thread stable writer identity through the Yjs `clientID`, persist it across sessions/peers, and resolve a selection's origin authors live. Highest fidelity (per-character, pre-commit, live) but a large new subsystem with taxonomy/persistence/GC surface. Deferred.
- **Full persistent blame gutter (deferred, NG2/§16).** A per-line author strip for the whole doc. Same blame source as v1 but a heavier, always-on UI; revisit once on-demand blame proves the source is trustworthy.

## 10) Decision log

- **D1:** Ship Route 1 (project-repo line blame + author recovery). Rationale: line-coherent source, correct identity via reused machinery, contained surface, no CRDT changes.
- **D2:** Reject shadow-repo blame as the source. Rationale: per-writer refs are not line-coherent (§8/§9), verified against `commitWipInner`.
- **D3:** Reuse the sibling attribution spec's author recovery for identity (trailer → `%an`). Rationale: single source of truth for "real author"; avoids a second, divergent recovery path. **Cross-spec dependency** (see §12/§14 R4).
- **D4:** Never fabricate an author (NFR1). Uncommitted / unresolvable → `unattributed`. Rationale: below-real fidelity must read as unknown, not a guess.
- **D5 (proposed):** Line granularity for v1; character/pre-commit granularity is Strategy B (deferred). Confirm with owner.
- **D6 (proposed):** **v1 is source-mode-only.** Verified: source mode maps a selection to a source line range exactly (CM binds to `Y.Text('source')`); WYSIWYG has no PM-position → source-offset map. Rationale: ship the exact, correct mapping now; don't gate v1 on the harder WYSIWYG problem. WYSIWYG blame → Q6 / §15b / §17. **Confirm with owner** (acceptable to launch blame in source mode only?).
- **D7 (proposed):** Add **UX-B (author-filter blame overlay)** as an alternative v1 shape sharing the UX-A backend. Rationale: the author-list half is free (doc-level contributors already exist) and the highlight half is pure source-mode line decoration — arguably a friendlier v1 than the selection-popover. Whether UX-B is the *primary* v1, a companion, or a fast-follow is **Q7 (owner call).**

## 11) Open questions

- **Q1 (Product, P1):** Multi-author range display — list all distinct authors with per-author line counts, or name the dominant author + "+N others"? *(Default: list distinct authors, most-recent-first, with counts.)*
- **Q2 (Product, P1):** Invocation gesture — selection toolbar button, right-click context action, command-palette entry, or keyboard shortcut? (Which fits the editor's existing selection affordances?) *(Investigate existing selection UI in `packages/app`; bring back a recommendation.)*
- **Q3 (Technical, P0):** The **byte-mapping is verified** (§12) — a client-local line-range computation from `Y.Text('source')` is arithmetically sound, no round-trip needed for the mapping. Residual: does the client need any server signal to know how far the *committed* file lags the live source (for the `working-copy` boundary)? **→ Spike (§15).**
- **Q4 (Technical, P1):** Should blame reflect the **committed** file only, or also account for locally-modified-but-uncommitted lines (marking them `working-copy`)? *(Default: blame committed; mark the rest `working-copy` — FR4.)*
- **Q5 (Cross-cutting, P0):** Sequencing vs. the sibling attribution spec — does Route 1 **require** that spec's recovery to be merged first, or can it ship with a `%an`-only fallback and upgrade later? (See §14 R4.)
- **Q6 (Product + Technical, P0):** WYSIWYG-mode blame — is **source-mode-only** acceptable for v1 (D6)? If WYSIWYG must be covered, is **block-granular** blame (whole block/paragraph attribution) acceptable, or is sentence precision required (→ Strategy B, NG1)? Block-granular needs its own spike (§15b) to confirm parse-time position attrs survive editing usefully. *(Default: source-mode-only v1; WYSIWYG block-granular as fast-follow.)*
- **Q7 (Product, P0):** Which v1 UX shape — **UX-A** (select → blame popover), **UX-B** (author panel → click-to-highlight, §9), or **both**? UX-B's author-list half is free (doc-level contributors) and its highlight half is pure source-mode decoration; it may be the friendlier primary v1. *(Default: build the shared backend once; ship UX-B author-list immediately, then whichever interaction the owner prefers.)*

## 12) Assumptions

- **VERIFIED (source-level):** on-disk bytes are `Y.Text('source').toString()` **verbatim, frontmatter included**, so line *N* in source === line *N* on disk with **no bespoke frontmatter offset**. Evidence: the persistence write derives `markdown = prependFrontmatter(stripFrontmatter(ytext))` and writes it via `tracedWriteFile` (`persistence.ts` ~L1358–1367, L1834); `stripFrontmatter`/`prependFrontmatter` are an exact identity round-trip (`match[0] + slice(match[0].length)`; `core/src/extensions/frontmatter.ts`). The pre-write `normalizeBridge` compare is a skip-if-unchanged detector, never applied to written bytes. The residual to confirm in the spike is *liveness*, not the mapping: debounce lag, committed-vs-working-copy, and CRLF/BOM line-counting parity (below).
- The project repo is a real git repo with the doc's file committed (the common shared-repo case); non-repo / uncommitted paths are handled by FR4.
- The sibling spec's `ok-contributors:` trailer parser is reusable as the identity source (D3) — a real cross-spec dependency, not an inline reinvention.
- The existing `contentRoot` + path→docName helper and realpath-escape guard are sufficient for safe path resolution (NFR3).

## 13) In scope (implement now)

- `selectionToLineRange` client helper — **source mode only** (FR1; D6) + the source-mode liveness spike (§15a). WYSIWYG affordance gated off in source-mode-only v1.
- `blameRange` server helper + author recovery integration (FR2, FR3, FR4).
- `GET /api/blame` endpoint + `BlameSuccessSchema` colocated with `history.ts` (FR5) — range mode (UX-A) and full-doc mode (UX-B) share the schema.
- **UX-A:** selection "Blame" affordance + popover + jump-to-commit (FR6, FR7); degradation surface (FR4).
- **UX-B:** per-file Authors panel from doc-level contributors (FR8 — free, both modes); author-click → source-mode line-range highlight (FR9). *Exact v1 interaction split pending Q7 — the shared backend is in scope regardless.*
- Tests (§ below).

## 14) Risks & mitigations

- **R1a — WYSIWYG selection has no source offset (the real Step-1 risk):** verified — no PM-position → source-offset map exists; the only primitive re-serializes the slice to text. Sentence-precise WYSIWYG blame is infeasible with today's primitives. *Mitigation:* v1 source-mode-only (D6); WYSIWYG → block-granular (own spike §15b) or deferred (§17). This displaces the frontmatter-offset concern as the top Step-1 risk.
- **R1b — Range→line drift, source mode (narrowed):** byte-mapping verified (§12), so frontmatter offset is a non-risk. Residual is purely *liveness* — the committed file the blame reads can trail the live source (debounce + commit lag), and CRLF/BOM can skew line counting. *Mitigation:* spike the lag/parity dimension (§15); mark uncommitted lines `working-copy` rather than mis-blaming onto the neighbor commit.
- **R2 — Wrong author on `Auto-save`/merge commits:** raw blame names the sync committer. *Mitigation:* FR3 author recovery + `--no-merges`/last-author rule (sibling spec D2).
- **R3 — Path-safety / injection:** `docName`/line args reaching `git blame`. *Mitigation:* reuse path→docName + realpath-escape guard; validate line args as positive integers; reject synthetic/out-of-`contentRoot` docs (NFR3).
- **R4 — Cross-spec coupling:** identity correctness depends on the sibling attribution spec (D3/Q5). *Mitigation:* decide sequencing (Q5); ship with `%an` fallback if recovery lands later, upgrading identity transparently.
- **R5 — Cardinality leak:** identities/paths in telemetry. *Mitigation:* identities in response body only; spans use `doc.name` + bounded counters (NFR2, STOP rule).

## 15) Spike (de-risk before build)

**Byte-mapping is already verified at source level** (§12): disk === `Y.Text('source')` verbatim, frontmatter included, no offset. So the spike is **re-scoped to the liveness/parity dimension only** — the mapping arithmetic itself needs no proof.

**§15a — Source-mode liveness spike (required for v1).** Confirm the *committed-file* the blame reads stays line-aligned with a source-derived range under realistic lag. In a running dev editor: (a) select a committed passage, derive the line range from `Y.Text('source')`, assert `git blame -L` names the expected line author; (b) type into the passage without waiting for persistence/commit, assert those lines resolve to `working-copy`/`unattributed` (not mis-blamed onto the neighbor commit); (c) a CRLF/BOM doc — assert client line counting matches git's. Resolves Q3/Q4 and R1b with real execution.

**§15b — WYSIWYG block-granular spike (only if Q6 wants WYSIWYG in v1).** Determine whether a WYSIWYG selection's covering top-level block node(s) expose a usable source line range: inspect `position-slice`/`sourceRaw` node attrs, edit the block, and check whether the stored source position stays correct or goes stale. Verdict decides whether WYSIWYG block-granular blame is buildable now or defers to Strategy B (§17). Skip if D6 (source-mode-only) is accepted.

Throwaway code; keep the findings. (Skill: `[public/open-knowledge/.agents/skills/spike/SKILL.md](../../public/open-knowledge/.agents/skills/spike/SKILL.md)`.)

## 16) Test plan

- **Unit:** `blameRange` over a fixture project repo — multi-commit, multi-author, a rename, and a merge; assert per-line `{sha,author}` and boundary/uncommitted → empty. `selectionToLineRange` over docs with/without frontmatter.
- **Author recovery:** a fixture `Auto-save:` commit with an `ok-contributors:` trailer surfaces the content author, not the committer; a trailer-less commit falls back to `%an` (mirrors the sibling spec's recovery tests).
- **Integration (regression = the journey):** in `packages/app/tests/integration`, seed a repo, commit a passage as author X, select it in-editor, call `GET /api/blame`, assert the span names X (not the sync committer); a just-typed uncommitted selection returns `unattributed`/`working-copy`.
- **Safety:** out-of-`contentRoot` / synthetic docName and malformed line args are rejected (NFR3).
- **Gate:** `cd public/open-knowledge && bun run check`.

## 17) Future work

- **WYSIWYG-mode blame** (D6/Q6): block-granular first (map selection → covering block → source range via parse-time position attrs, pending §15b), then sentence-precise via Strategy B.
- **Strategy B — CRDT per-range provenance** (NG1): per-character, pre-commit, live authorship by threading writer identity through the Yjs `clientID`. Also the path to sentence-precise WYSIWYG blame.
- **Persistent blame gutter** (NG2): always-on per-line author strip once on-demand blame proves the source.
- **"Who + when → why":** deeper linking from a blame span into the full commit context (diff, agent summary, Timeline thread).
