---
name: trello-use
description: "**MANDATORY prerequisite** — load this BEFORE calling any Trello MCP tool (`trelloRead*`, `trelloWrite*`, `trelloSearch`). Covers the ARI id format every tool requires and the cross-cutting rules (current-user lookup, UTC date handling, Inbox vs boards, ordered creation, pagination). Skipping it causes malformed-id errors and wrong due-date/timezone results."
disable-model-invocation: false
---

# Trello MCP Server — Usage Skill

Trello MCP tools are action-dispatched (`action: "..."` selects the operation) and validate strictly: a field not used by the chosen action is rejected, not ignored.

## 1. Identify the current user first

Call `trelloReadMember` with `action: "get_me"` to get the authenticated user's profile, including `prefs.timezone`. Do this before any relative-time request ("today", "this week") for cards, Inbox, and search — most concretely, before `trelloWriteCard`/`trelloWriteInbox` `create` or `update` when the user gives a `due` date/time in local or relative terms (e.g. "tomorrow 1pm"): resolve `prefs.timezone` first, convert to UTC, then pass that as `due`. (Planner events use a different timezone source — see §3.)

## 2. ARI id format (critical — read before passing any id)

Every id-typed parameter (`cardId`, `boardId`, `listId`, `workspaceId`, `checklistId`, `itemId`, `labelId`, `plannerCalendarId`, `providerAccountId`) is an ARI, not a raw Trello object id. Most are workspace-scoped and follow this shape:

```
ari:cloud:trello::<entity>/workspace/<workspaceObjectId>/<entityObjectId>
```

The two exceptions (last two rows below) are not workspace-scoped: planner calendar has no workspace segment, and a third-party account is a different ARI namespace.

| Entity | Shape |
|---|---|
| board | `ari:cloud:trello::board/workspace/<workspaceObjectId>/<boardObjectId>` |
| card | `ari:cloud:trello::card/workspace/<workspaceObjectId>/<cardObjectId>` |
| list | `ari:cloud:trello::list/workspace/<workspaceObjectId>/<listObjectId>` |
| checklist | `ari:cloud:trello::checklist/workspace/<workspaceObjectId>/<checklistObjectId>` |
| check item | `ari:cloud:trello::check-item/workspace/<workspaceObjectId>/<itemObjectId>` |
| label | `ari:cloud:trello::label/workspace/<workspaceObjectId>/<labelObjectId>` |
| workspace | `ari:cloud:trello::workspace/<workspaceObjectId>` |
| planner calendar | `ari:cloud:trello::planner-calendar/<calendarObjectId>` (no workspace segment) |
| third-party account | `ari:third-party:<provider>::account/<providerAccountId>` (e.g. Google) |

Rules:
- **Always get the ARI from a read tool first**, never construct or guess one. `trelloWriteCard`, `trelloWriteChecklist`, `trelloWriteInbox`, and `trelloWritePlanner` all reject Trello URLs/short links on their id params ("Do NOT pass a Trello URL or short link; use `trelloReadCard`/`trelloReadInbox` first to obtain the ARI").
- **Two read `get` actions also accept a plain Trello URL**, as a convenience: `trelloReadBoard` (`get`) takes a board URL (`https://trello.com/b/<shortLink>/...`) and `trelloReadCard` (`get`) takes a card URL (`.../c/<shortLink>/...`), in addition to an ARI. Every other tool — and all write tools — take ARIs only.

## 3. Dates are always UTC ISO 8601

`due` (trelloWriteCard, trelloWriteInbox) and `start`/`end` (trelloWritePlanner) are UTC ISO 8601 (`"2026-06-01T17:00:00.000Z"`). For local/relative times, convert using:
- Cards & Inbox: `trelloReadMember` (`get_me`) → `prefs.timezone`
- Planner events: `trelloReadPlanner` (`get`) → `primaryCalendar.timezone`
- No time given → default to 09:00 local.

`trelloSearch` due-date qualifiers (`due:`) use the same UTC-boundary rule — resolve `prefs.timezone` first.

## 4. Inbox is not a Trello board

Inbox is a personal quick-capture surface with its own tools: `trelloReadInbox` / `trelloWriteInbox`. Don't try to access it via `trelloReadBoard`/`trelloWriteBoard` — go through the Inbox-specific tools instead. `trelloWriteInbox` `create` takes no `pos` — new Inbox cards always land at the top.

## 5. Ordered creation

When creating multiple items that must keep a specific order — lists, cards, checklists on a card, or check items in a checklist — set `pos` to distinct sequential integers (`1, 2, 3, …`) in the intended order. Scope is per parent: cards per list, lists board-wide, checklists per card, check items per checklist. Distinct values fix the order regardless of which call lands first, so you can create them in parallel. `pos` also accepts `"top"`/`"bottom"` where the field is exposed.

**Example — 3 lists on a new board, order matters (`To Do` → `In Progress` → `Done`):**
```json
[
  { "tool": "trelloWriteList", "action": "create", "boardId": "ari:cloud:trello::board/workspace/<workspaceObjectId>/<boardObjectId>", "name": "To Do", "pos": 1 },
  { "tool": "trelloWriteList", "action": "create", "boardId": "ari:cloud:trello::board/workspace/<workspaceObjectId>/<boardObjectId>", "name": "In Progress", "pos": 2 },
  { "tool": "trelloWriteList", "action": "create", "boardId": "ari:cloud:trello::board/workspace/<workspaceObjectId>/<boardObjectId>", "name": "Done", "pos": 3 }
]
```
All three can be issued in parallel — `pos` is per board, and distinct values (1/2/3) fix the order no matter which call lands first.

**Example — 3 cards in one list, order matters (`Step 1` → `Step 2` → `Step 3`):**
```json
[
  { "tool": "trelloWriteCard", "action": "create", "listId": "ari:cloud:trello::list/workspace/<workspaceObjectId>/<listObjectId>", "name": "Step 1", "pos": 1 },
  { "tool": "trelloWriteCard", "action": "create", "listId": "ari:cloud:trello::list/workspace/<workspaceObjectId>/<listObjectId>", "name": "Step 2", "pos": 2 },
  { "tool": "trelloWriteCard", "action": "create", "listId": "ari:cloud:trello::list/workspace/<workspaceObjectId>/<listObjectId>", "name": "Step 3", "pos": 3 }
]
```
`pos` here is scoped to that one list — a card created with `pos: 1` in a different list is independent of this list's `pos: 1`.

**Counter-example — order doesn't matter:** just create the item without a `pos` (or with `"bottom"`). Don't invent sequential numbers when the user hasn't asked for a specific order — that's unrequested precision.

## 6. Pagination

Read tools that list collections use cursor pagination (`cursor` in, `pageInfo.endCursor`/`nextCursor` + `hasNextPage`/`hasMore` out). Keep paginating while `hasNextPage`/`hasMore` is true instead of assuming the first page is complete — this matters especially for `trelloReadBoard` `list`/`list_by_workspace` when filtering by `visibility` (filtered server-side per page, not globally before pagination).
