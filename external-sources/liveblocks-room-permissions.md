---
title: Liveblocks — room permissions (source)
description: "Captured reference: Liveblocks room permission values (write/read/presence:write) and the defaultAccesses/groupsAccesses/usersAccesses model, access tokens vs ID tokens."
type: source
source_url: https://liveblocks.io/docs/rooms/permissions
fetched: 2026-07-06
tags:
  - source
  - liveblocks
  - permissions
  - view-only
---
# Liveblocks — room permissions (source)

> Raw reference captured from Liveblocks docs (<https://liveblocks.io/docs/rooms/permissions>) on 2026-07-06. Preserved to ground [[guides/view-only-access|the view-only access research]]. Complements the deeper [[guides/live-blocks|Liveblocks case study]].

## Permission values

- **`room:write`** — full access: view **and** edit the room.
- **`room:read` + `room:presence:write`** — view the room's storage (read-only) **while still editing your own presence**. This is the load-bearing pattern: a read-only client can still **broadcast its cursor/presence** even though it cannot mutate storage (documented in the [[guides/live-blocks|Liveblocks case study]]).

## Where permissions are set

- **`defaultAccesses`** — default permission for the entire room.
- **`groupsAccesses`** — per-group permission.
- **`usersAccesses`** — per-user permission. Each narrower level can grant access even when `defaultAccesses` is empty (private room + explicit user allow).

## Group- & workspace-level permissions

> Additional detail from the Liveblocks guide *Managing rooms, users, and permissions* (<https://liveblocks.io/docs/guides/managing-rooms-users-permissions>), captured 2026-07-06.

- **Groups (`groupsAccesses`)** — a group is a custom string id (e.g. `"engineering"`). Users are assigned groups at auth time (`identifyUser({ userId, groupIds: ["engineering"] })`), and a room grants the whole group access in one entry:

  ```ts
  groupsAccesses: { engineering: ["*:write"] }  // whole team can view + edit
  ```

  Revoke by setting the group to `null`; change it live via `liveblocks.updateRoom()`. This is how a **team / workspace** gets read-only (`*:read`) or edit access to a room without listing every user individually.
- **Workspace / organization (`organizationId`)** — pass `organizationId` at `identifyUser` to scope a user (and the resources they create) to one workspace, so access can't cross org boundaries.
- **Precedence:** `usersAccesses` › `groupsAccesses` › `defaultAccesses` — each narrower level overrides the wider one. A private room (empty `defaultAccesses`) can still grant a specific group or user access.

**For OK view-only:** `groupsAccesses` is the model for "this whole workspace/team can *view* this doc" — a group-scoped read grant. It maps onto OK's open question of where roles should live (a federated store could own group→role like Liveblocks, vs the per-user GitHub access OK relies on today). See [[guides/view-only-access|the view-only research]].

## Access tokens vs ID tokens

- **Access tokens** — permissions embedded directly in the token; the token is the source of truth (custom permission management).
- **ID tokens (recommended for production)** — permissions live in the Liveblocks backend and are "checked at the door" each time a user enters a room; the room is the source of truth.