---
title: Google Docs — sharing roles & view-only (source)
description: "Captured reference: Google Drive/Docs permission roles (Viewer, Commenter, Editor, Owner) and the viewer download/print/copy restriction toggle."
type: source
source_url: https://support.google.com/docs/answer/2494822
fetched: 2026-07-06
tags:
  - source
  - google-docs
  - permissions
  - view-only
---
# Google Docs — sharing roles & view-only (source)

> Raw reference captured from Google Docs Help (<https://support.google.com/docs/answer/2494822>) on 2026-07-06. Preserved to ground [[guides/view-only-access|the view-only access research]].

## Permission roles (My Drive files)

- **Viewer** — read-only. Can view and (by default) download; **cannot** comment, edit, or share.
- **Commenter** — can view, comment, and (by default) download; **cannot** edit or change sharing.
- **Editor** — view, comment, edit, download, share; cannot change ownership.
- **Owner** — full control including permission management and ownership transfer.

## Viewer/commenter download restriction

> "To give viewers and commenters the option to download, print, and copy a file, owners can click Share > Settings and check the box."

By default this box is **off** — i.e. viewers/commenters are restricted from downloading/printing/copying unless the owner explicitly enables it. A meaningful hardening of read-only beyond just "can't type."

## Sharing surfaces

- **Specific people** — add individual emails, each assigned a role.
- **Anyone with the link** — public link sharing at a chosen role (e.g. anyone-with-link = Viewer).
- **Published to the web** — a separate read-only published view (distinct from edit access).

## Enforcement

Viewers are prevented from editing by role-based restrictions built into the app UI — the edit affordances are simply absent for viewer accounts (server-enforced, not just hidden).