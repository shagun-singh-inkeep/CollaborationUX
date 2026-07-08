---
title: Notion — sharing & permissions (source)
description: "Captured reference: Notion permission levels (Full access, Can edit, Can comment, Can view), guests vs members vs teamspaces, and public web links."
type: source
source_url: https://www.notion.com/help/sharing-and-permissions
fetched: 2026-07-06
tags:
  - source
  - notion
  - permissions
  - view-only
---
# Notion — sharing & permissions (source)

> Raw reference captured from Notion Help (<https://www.notion.com/help/sharing-and-permissions>) on 2026-07-06. Preserved to ground [[guides/view-only-access|the view-only access research]].

## Permission levels (per page)

- **Can view** — "read the content on the page, but they won't be able to comment, edit, or share." (Notion's view-only is stricter than Google's — **no commenting** at this tier.)
- **Can comment** — view + comment; cannot edit or share.
- **Can edit** — modify content; cannot share.
- **Full access** — edit any content **and** share with anyone.

## Access types

- **Members vs guests** — members are workspace participants; guests are external people invited to specific pages by email.
- **Teamspaces** — default teamspaces grant all workspace members access; restricted teamspaces require explicit invites with an assigned tier.
- **Public web links** — "Anyone on the web with link" allows viewing without a Notion account; commenting/editing still requires login.

## Broadest-access rule

Notion grants "the broadest level of access given to a user" — if a user already has Full access via a parent/teamspace, a narrower per-page setting won't reduce it. (Relevant to inheritance/enforcement design.)

## Notes

- Export/duplicate restrictions for view-only are not specified on this page.