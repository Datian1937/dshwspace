# Issue tracker: GitHub

Issues and specs for this repo live as GitHub issues in `Datian1937/dshwspace`.

## How to work with tickets

Use the MattSkillsDeck tools. They talk to the configured GitHub backend, read back what they wrote, and report honestly when a response is partial. Do not hand-write tracker commands; the only work outside the tools is signing in (`gh auth login`) and creating the remote repo.

- Create a ticket → `deck_issue_create`
- Read a ticket → `deck_issue_get`
- See maps and open tickets → `deck_context`
- See a map's children and progress → `deck_map_snapshot`
- Thin rows by state, label, or assignee → `deck_issue_list`
- Create a whole map (map + children + edges) → `deck_map_plan_create`
- Add a parent or blocking edge → `deck_map_link`
- Edit a ticket (comment / labels / assignee / close / body) → `deck_issue_patch`

If a tool fails, read the receipt's detail JSON: `status` (`ok` / `partial` / `unsupported`) and `reason` say whether to retry, wait for quota, or fall back. After a write timeout, read the ticket back before retrying; the write may already have landed.

## Labels

Every new ticket carries at least one label from the mandatory set: `bug`, `needs-triage`, `wayfinder:grilling`. `deck_issue_create` fills a required label in when none is passed. The full label vocabulary, including the five triage roles and the wayfinder ticket types, is in `docs/agents/triage-labels.md`.

## Ticket body conventions

Write the body to a file with real newlines, then hand that file to the tool. Never inline the body into a command line, never write a literal `\n` two-character sequence, and never start the body with a BOM. Read the ticket back after writing.

Sections in order: `## 目的`, `## 维护者原话` (only when there is a verbatim quote), `## 边界`, `## 做完`, any optional sections, and `## 进度` last, with the newest entry first. Progress entries are written with `progress: {percent, note}`; the tool stamps the time.

## Pull requests as a triage surface

**PRs as a request surface: no.** _(Set to `yes` if this repo treats external PRs as feature requests; `/triage` reads this flag.)_

## When a skill says "publish to the issue tracker"

Create a GitHub issue with `deck_issue_create`.

## When a skill says "fetch the relevant ticket"

Read it with `deck_issue_get`: body, comments, and labels.

## Wayfinding operations

Used by `/wayfinder`. The **map** is a single issue labelled `wayfinder:map`; **child** issues are tickets.

- **Map**: create it with `deck_map_plan_create`.
- **Child ticket**: attach it with `deck_map_link` (`parentKey`); label it `wayfinder:<type>` (`research` / `prototype` / `grilling` / `task`).
- **Blocking**: `deck_map_link` with `blockedBy` (the tickets that block this one). GitHub's native issue dependencies are used when available; only when they are not does the tool fall back to a `Blocked by: #n` line in the child body.
- **Frontier query**: `deck_map_snapshot` over the map's open children; drop blocked and assigned tickets; first in map order wins.
- **Claim**: assign the ticket to the current user with `deck_issue_patch`; this is the session's first write.
- **Resolve**: comment the conclusion, close the ticket, then add a pointer to the map's Decisions-so-far.
