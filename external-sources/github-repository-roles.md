---
title: GitHub — repository roles & the REST permissions object (source)
description: "Captured reference: GitHub's five repository roles (Read/Triage/Write/Maintain/Admin) and the REST 'Get a repository' permissions object (admin/maintain/push/triage/pull) used to detect the authenticated user's read vs write access."
type: source
source_url: https://docs.github.com/en/organizations/managing-user-access-to-your-organizations-repositories/managing-repository-roles/repository-roles-for-an-organization
source_url_2: https://docs.github.com/en/rest/repos/repos#get-a-repository
fetched: 2026-07-06
tags:
  - source
  - github
  - permissions
  - view-only
---
# GitHub — repository roles & the REST permissions object (source)

> Raw reference captured 2026-07-06 from GitHub Docs: [repository roles for an organization](https://docs.github.com/en/organizations/managing-user-access-to-your-organizations-repositories/managing-repository-roles/repository-roles-for-an-organization) and the REST [Get a repository](https://docs.github.com/en/rest/repos/repos#get-a-repository) endpoint. Preserved to ground [[guides/view-only-access|the view-only access research]].

## The five repository roles

- **Read** — "non-code contributors who want to view or discuss your project." Can pull/clone; **cannot push.**
- **Triage** — manage issues/PRs/discussions **without write access.** Still cannot push code.
- **Write** — "contributors who actively push to your project." **First level that can push and merge.**
- **Maintain** — manage the repo without destructive/admin actions. Includes push.
- **Admin** — full control incl. destructive actions. Includes push.

Roles are assigned the same way to organization members, outside collaborators, and teams.

**The dividing line for editing is push:** Read + Triage are read-only for *code/content*; Write + Maintain + Admin can push.

## The REST `permissions` object

`GET /repos/{owner}/{repo}` returns, for the **authenticated user**, a `permissions` object of booleans:

- `admin` — administrative actions
- `maintain` — manage without destructive actions
- `push` — **can push code changes** (the write bit)
- `triage` — manage issues/PRs
- `pull` — **read-only access** (basic repository access)

Higher tiers imply lower ones (an `admin` user has all bits). To decide read vs write for a document editor:

- **Read-only (viewer):** `pull: true`, `push: false`.
- **Editor:** `push: true` (equivalently `push || maintain || admin`).

This is the exact signal OK already fetches in `github-permissions.ts` (it reads `permissions.push`).