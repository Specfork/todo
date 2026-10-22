# Tasks

**Status:** Draft · **Prefix:** TASK

## Purpose

The task record, its numbering, completion and undo, trash, bulk operations, and labels.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| TASK-1 | Task: `{id, project_id, index, identifier, title (1–250 chars, trimmed), description (rich text, ≤ 65,535 chars), done (default false), done_at?, due_date?, start_date?, end_date?, priority, percent_done, hex_color?, repeat_after, repeat_mode, repeat_months, is_favorite, labels[], reminders[], related_tasks{}, created, updated, deleted_at?}`. | create `{title: "Pay rent"}` → `done` false, `priority` 0 |
| TASK-2 | `priority`: 0 none (default), 1 low, 2 medium, 3 high, 4 urgent, 5 do_now; other values → `invalid`. `percent_done`: number 0–1 (default 0), else `invalid`. | `priority: 6` → `invalid`; `percent_done: 1.5` → `invalid` |
| TASK-3 | `due_date`, `start_date`, `end_date` are optional instants (date-only input per SET-7). `end_date` before `start_date` when both are set → `invalid`; `due_date` is independent of both. | start 10th, end 9th → `invalid` |
| TASK-4 | `index` comes from a per-project counter starting at 1 that never goes back, even after deletes. `identifier` = project identifier + `-` + index, or `#` + index when the project has none. A task can be read by `(project_id, index)`. | 3rd task in `HOME` → `HOME-3` |
| TASK-5 | Create without `project_id` uses `default_project_id`. A new task lands in every view of its project ([VIEW-6](views.md), [VIEW-10](views.md)). | create without project → in Inbox |
| TASK-6 | Update is partial: only sent fields change; `null` clears an optional field. Clients cannot set `index`, `identifier`, `done_at`, `created`, `updated`, `deleted_at`. | send `{priority: 3}` → title unchanged |
| TASK-7 | Moving to another own project (change `project_id`) gives the next index there; the old `(project_id, index)` still resolves to the task. The task goes into each target kanban view's done bucket if done, else its default bucket. | move `HOME-3` to `WORK` → `WORK-8`; `HOME-3` still opens it |
| TASK-8 | Done: `false → true` sets `done_at` to the server's now; `true → false` clears it. Repeating tasks follow [REP-2](recurrence.md) instead. Completing a parent does not complete subtasks and vice versa. | mark done → `done_at` set; reopen → `done_at` null |
| TASK-9 | Every done change returns `undo_token`, single-use, valid 60 s. `undo(undo_token)` restores `done`, `done_at`, dates, reminders, buckets and positions to their state before the change. Used or expired → `gone`. | complete repeating task, undo within 60 s → old due date back |
| TASK-10 | Delete moves a task to trash: `deleted_at` = now; it disappears from projects, views, filters, search, reminders and the digest. The trash lists trashed tasks newest first. | delete → task not in list view |
| TASK-11 | Restore from trash clears `deleted_at`; the task returns to its project (or `default_project_id` if that project is gone or archived), at the top of each view and in each kanban default bucket. Permanent removal happens on request (`purge`) or 30 days after `deleted_at` (checked at least hourly) and removes its label links, relations, reminders and notifications. | trashed 31 days → gone; restore → `not_found` |
| TASK-12 | Duplicate copies all fields except `done` (false), `done_at` (null) and `index` (new); copies labels and reminders, places the copy at the top, and adds `copied_from` to the original ([REL-1](subtasks.md)). | duplicate done task → copy not done |
| TASK-13 | Bulk update `{task_ids (1–100, duplicates ignored), fields}`: `fields` may set `title`, `description`, `done`, `due_date`, `start_date`, `end_date`, `priority`, `percent_done`, `hex_color`, `repeat_after`, `repeat_mode`, `repeat_months`, `project_id`, `bucket_id`, `is_favorite`, `labels`, `reminders`; any other key → `invalid`. All or nothing: one unknown or archived task → nothing changes. | 3 ids, one foreign → `not_found`, none changed |
| TASK-14 | Bulk create: 1–100 tasks in one project, all or nothing; a failing item → `invalid` naming its 0-based position. | 101 tasks → `invalid` |
| TASK-15 | Label: `{id, title (1–250 chars), description?, hex_color?, created, updated}`; titles need not be unique. | two labels `home` → both stored |
| TASK-16 | Adding a label already on the task → `conflict`; removing one not on it → `not_found`. Setting `labels` replaces the whole set. Any label change updates the task's `updated`. Deleting a label removes it from every task; the tasks stay. | add `home` twice → `conflict`; delete label → tasks keep other labels |

## Out of scope

- Assignees, comments, mentions, reactions, attachments and cover images, time tracking, task subscriptions, read/unread markers.
- Task identifiers across instances; CalDAV access.

## Decisions

- Title capped at 250 chars and descriptions at 65,535; the source left both unbounded.
- `priority`, `percent_done` and date order are validated; the source accepted any number and any order.
- Update is partial; the source cleared every field the client left out.
- Trash with restore (TASK-11); the source kept deleted tasks 30 days but offered no restore.
- Undo is a server token (TASK-9); the source's client re-sent the opposite done value, which re-advanced repeating tasks.
