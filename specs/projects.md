# Projects

**Status:** Draft · **Prefix:** PRJ

## Purpose

Projects hold tasks. They nest into a tree, can be archived, duplicated, favorited and deleted.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| PRJ-1 | Project: `{id, title (1–250 chars, trimmed), description (rich text, ≤ 65,535 chars), identifier? (0–10 chars of `[A-Z0-9]`, stored upper-case), hex_color?, parent_id?, position (number), is_archived (default false), is_favorite (default false), created, updated}`. | create with title `"  "` → `invalid` |
| PRJ-2 | `identifier` is unique among the owner's projects, ignoring case → `taken`; empty means none. | `home` when `HOME` exists → `taken` |
| PRJ-3 | `hex_color` (projects, tasks, labels, views): 6 hex digits; one leading `#` is stripped; stored lower-case; anything else → `invalid`; empty clears it. | `#FF8800` → `ff8800` |
| PRJ-4 | Nesting has no depth limit. `parent_id` must be an own existing project; self → `invalid`; a parent inside the project's own subtree → `cycle`. Moving a project moves its whole subtree. | A ⊃ B ⊃ C, set A's parent to C → `cycle` |
| PRJ-5 | Siblings are ordered by `position` ascending, then `id`. A project created without `position` goes after the last sibling (last + 65,536). When two neighbours end up less than 0.01 apart, all siblings are renumbered evenly, keeping their order. | create 3 projects → positions ascending in creation order |
| PRJ-6 | Sign-up creates the project `Inbox` and sets it as `default_project_id`. The default must be an own, non-archived project, else `invalid`. A user always has a default: deleting it makes the oldest remaining non-archived top-level project the default, or a new `Inbox` if none is left. | delete Inbox, no other project → new Inbox is default |
| PRJ-7 | Archiving sets `is_archived` on the project and every descendant. An archived project and its tasks, views and buckets are read-only: any write → `archived`, except unarchiving. Unarchiving clears `is_archived` on the project and every descendant. Unarchiving a project whose parent is archived → `archived`. Creating a project under, or moving one into, an archived parent → `archived`. | archived project, rename task → `archived`; unarchive child of archived parent → `archived` |
| PRJ-8 | The default project cannot be archived, directly or by archiving an ancestor → `invalid`. | archive parent of Inbox → `invalid` |
| PRJ-9 | Project lists leave out archived projects unless `include_archived=true`. Tasks of archived projects are left out of saved filters, the agenda, the favorites list, reminders and the digest; they stay readable inside their project. | archived project, overdue task → not in digest |
| PRJ-10 | Delete is permanent and covers the project, all descendants, their tasks (trashed ones too), views, buckets and positions; labels stay. A preview call returns `{project_count, task_count}` that the delete would remove. | delete project with 2 children and 9 tasks → preview `{3, 9}` |
| PRJ-11 | Duplicate copies title + ` (copy)`, description, color, parent, views, buckets, tasks (fields, labels, reminders, positions, buckets), and relations between copied tasks; `identifier` is cleared, `is_favorite` false, child projects are not copied. Each copied task gets a `copied_from` relation ([REL-1](subtasks.md)) to its original. | duplicate → copy has no identifier, same task count |
| PRJ-12 | `is_favorite` marks a project; favorite non-archived projects are listed apart, ordered as in PRJ-5. | favorite then archive → not in favorites list |

## Out of scope

- Project backgrounds and stock-photo search, project sharing and links, per-project webhooks, subscriptions.

## Decisions

- Identifiers are unique per owner, not per instance: data sets are private (SET-2).
- A default project always exists; the source could leave none after the owner deleted it.
- Archiving an ancestor of the default project is refused; the source archived it anyway.
- A duplicated project's title gets ` (copy)`; the source used a different suffix.
