---
title: GitHub — listing repository collaborators & contributors (source)
description: "Captured reference: the two GitHub REST endpoints for enumerating people on a repo — List repository collaborators (requires write/maintain/admin; affiliation=all returns everyone with access) and List repository contributors (committers only; only the first 500 author emails link to GitHub users)."
type: source
source_url: https://docs.github.com/en/rest/collaborators/collaborators
source_url_2: https://docs.github.com/en/rest/repos/repos#list-repository-contributors
fetched: 2026-07-06
tags:
  - source
  - github
  - mentions
  - identity
  - phase-2
---
# GitHub — listing repository collaborators & contributors (source)

> Raw reference captured 2026-07-06 from GitHub Docs: the REST [List repository collaborators](https://docs.github.com/en/rest/collaborators/collaborators) and [List repository contributors](https://docs.github.com/en/rest/repos/repos#list-repository-contributors) endpoints. Preserved to ground [[guides/collaborator-identity|the collaborator identity research]] — specifically "where does the `@`-mention directory come from."

## List repository collaborators — `GET /repos/{owner}/{repo}/collaborators`

**Permission (verbatim):** *"The authenticated user must have write, maintain, or admin privileges on the repository to use this endpoint."*

So a **push-level** token is required to enumerate collaborators. A Read- or Triage-only (pull-only) user **cannot** call it — ties back to the push dividing line in [[external-sources/github-repository-roles|GitHub repository roles]].

**`affiliation` parameter** (filters the returned set):

- `outside` — all external collaborators of an organization-owned repository.
- `direct` — all collaborators with direct repository permissions, regardless of organization membership.
- `all` (**default**) — all collaborators the authenticated user can access.

**Who `affiliation=all` includes (verbatim):** *"outside collaborators, organization members that are direct collaborators, organization members with access through team memberships, organization members with access through default organization permissions, and organization owners."* Team members also include the members of child teams.

**Net:** `affiliation=all` is the full *"everyone with access to this repo"* roster — exactly a mention directory — but it is readable only by a write/maintain/admin token.

## List repository contributors — `GET /repos/{owner}/{repo}/contributors`

- **Returns:** contributors sorted by number of commits, descending. Committers only — **not** "people with access."
- **Permission:** the docs do not state an explicit permission requirement; the endpoint is callable on **public** repositories (authentication is not stated as mandatory). Response codes include `200` (has content), `204` (empty repo), `403`, `404`.
- **500-author cap (verbatim gist):** *"Only the first 500 author email addresses in the repository link to GitHub users. The rest will appear as anonymous contributors without associated GitHub user information."*
- **Caching:** may return data a few hours old (contributor data is cached).

**Net:** `contributors` needs no push and works anonymously on public repos, but it is *participation-limited* (only people who have committed) and capped at 500 linked authors — a narrower, read-only fallback to the collaborators roster.
