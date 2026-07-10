# Comment model — 2026-07-10 design exploration

**Provenance:** user design exploration (Shagun, 2026-07-10) — **design intent, not verified feasibility.** Verify the anchoring/storage and MCP-write feasibility in /spec. Raw source: the screenshots + notes in [Untitled](ideas) (this folder).

Supports [OUTCOMES](OUTCOMES). Captured here so the outcomes reference a synthesized model rather than re-reading raw screenshots.

## The hero flow (mockups 15a → 15b → 15c)

**15a — one actions cluster.** On a text selection, **Comment** sits next to **Ask AI** in the right-hand actions group, divider-separated from formatting (B / I / U / code / link). Rationale: "the two things you *do with* a selection sit together." Collapses to a bubble icon when space is tight. Shortcut ⌘⇧M.

**15b — one box, one switch (the send step).** A single comment box carries a **Team / AI toggle** that decides the recipient:
- Pointed at **AI** → the send button becomes **Send to AI**; the anchored text **plus your note become the instruction** — no separate "prompt" field.
- Pointed at **Team** → posts a normal human thread.
- "A comment is a comment… Same box, one switch." ⌘↵ to send.

**15c — what AI sends back (a diff you accept).** The agent replies **inside the thread** with a proposed change **rendered as a diff on the anchored text — not applied yet**:
- **Accept edit** → writes it into the doc as **one edit** (shows in edit history / undo stepper) **and resolves the comment**.
- **Discard** → keeps the thread open; the comment and the edit stay linked.
- In-thread attribution: "Alex · asked AI", then "Assistant · proposed an edit".

## Questions the exploration raises (from the raw notes)

1. **Ask-AI vs leave-comment** as the two selection affordances → resolved in the mockups as the unified composer above (→ outcome **C2**, item **CQ2**).
2. **Agent suggestion / approve-decline** — "are we supporting the ability for an agent to send a suggestion you can approve/decline?" The mockups answer *yes* via 15c. **Open** in OUTCOMES as **CQ1**: whether accept/discard is the default only for Ask-AI-in-comment, the model for *all* agent edits (which un-parks the parent's PQ3 gating), or coexists with O8 direct-act. User directive 2026-07-10: **keep this open.**
3. **Access tiers + GitHub** — "view only / comment only / edit access, and how does that work with github?" Recurs across captures → outcome **C5** + item **CX2**; threads have no git representation, so comment-only never conflicts with git.

## Competitive captures (reference only — table-stakes calibration)

- **Notion** — comment on selection; comment/AI both reachable from the same selection surface.
- **Mintlify / Coda** — expandable comments section; docs-native threads.
- **Claude design** — selection → comment/refine affordances.
- **GitHub PR "stacked markdown comments"** — line-anchored review threads on markdown; closest git-native prior art, but comments live in the PR layer, not the file — mirrors OK's "threads never travel through git" boundary.

Full table-stakes / differentiators / deliberately-avoid synthesis is inherited from [../human-collaboration-foundations/evidence/prior-art-collaboration.md](../human-collaboration-foundations/evidence/prior-art-collaboration.md).
