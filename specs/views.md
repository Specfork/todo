# Views

**Status:** Draft · **Prefix:** VIEW

## Purpose

Ways to look at a project's tasks (list, table, kanban, calendar), kanban buckets, manual order, and the cross-project agenda.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| VIEW-1 | View: `{id, project_id, title (1–250 chars), kind: list\|table\|kanban\|calendar, position, filter? ([QRY-1](query.md)), bucket_mode: none\|manual\|filter, bucket_filters: [{title, filter}] (≤ 50), default_bucket_id?, done_bucket_id?}`; a project's views are listed by `position`. | kind `gantt` → `invalid` |
| VIEW-2 | A new project gets four views: `List` (list, 100, filter `done = false`), `Calendar` (calendar, 200), `Table` (table, 300), `Board` (kanban, 400, `manual`) with buckets `To do` (100, default bucket), `Doing` (200) and `Done` (300, done bucket). | new project → 4 views, board has 3 buckets |
| VIEW-3 | Non-kanban views always have `bucket_mode` `none`; a kanban view sent with `none` gets `manual`. In `filter` mode every `bucket_filters` item needs a valid filter; filters are checked on save → `invalid`. | list view with `manual` → stored `none` |
| VIEW-4 | A view shows its project's tasks matching the view filter AND the request filter. Deleting a view deletes its buckets and positions, not tasks. | delete board → tasks unchanged |
| VIEW-5 | Bucket: `{id, view_id, title (1–250 chars), limit (integer ≥ 0, 0 = none), position, count}`, listed by `position`; `count` = tasks in it matching the view filter. A view holds at most 100 buckets → `limit_reached`. | 101st bucket → `limit_reached` |
| VIEW-6 | In a `manual` kanban view every task of the project sits in exactly one bucket. A new or moved-in task goes to the requested bucket, else `default_bucket_id`, else the lowest-position bucket (ties: lowest id); a done task goes to the done bucket when set. | create task, no bucket given → `To do` |
| VIEW-7 | Moving a task into a different bucket whose `count` ≥ `limit` (limit > 0) → `bucket_full`; reordering inside a bucket is always allowed. | bucket limit 2 holding 2, move third in → `bucket_full` |
| VIEW-8 | Moving a task into the done bucket marks it done ([TASK-8](tasks.md), [REP-2](recurrence.md)) and moves it to the done bucket of every other manual kanban view of the project; moving it out marks it not done. Creating a task in the done bucket creates it done. Marking a task done elsewhere moves it to each view's done bucket, if set; marking it not done moves it from a done bucket to that view's default bucket and leaves other buckets alone. | drag to `Done` → `done` true, `done_at` set; reopen task in `Done` → back in `To do` |
| VIEW-9 | `default_bucket_id` and `done_bucket_id` must be buckets of the same view → `invalid`. Deleting a view's last bucket → `invalid`. Deleting a bucket moves its tasks to the default bucket (or the lowest remaining one if it was the default) and clears `default_bucket_id`/`done_bucket_id` that pointed to it. | delete `Doing` with 3 tasks → 3 tasks in `To do` |
| VIEW-10 | Manual order: each (task, view) pair has a numeric `position`; lists and buckets sort by it ascending, then `id`. New tasks go above the current top. A move sets the midpoint of its new neighbours; when a gap falls below 0.01 the view's positions are renumbered evenly across 0–2³², keeping order. | drop task between positions 1 and 2 → 1.5 |
| VIEW-11 | `filter` bucket mode: buckets are computed, one per `bucket_filters` item in order, each showing tasks matching the view filter AND its own filter; a task may appear in several; moving tasks between them → `invalid`. | filters `priority >= 3`, `priority < 3` → split by priority |
| VIEW-12 | Calendar view: `from`/`to` (default: the current local month in `time_zone`); a task appears when its span [`start_date` or `due_date`, `end_date` or `due_date`] meets [from, to); tasks without any date are left out unless `show_undated=true`. Sorted by span start, then `id`. | task Jun 3–5, window June → shown on 3, 4, 5 |
| VIEW-13 | Agenda (across all own non-archived projects): not-done tasks with `due_date` in [`from`, `to`), defaults now and now + 7 days; `show_overdue` (default false) adds `due_date` < `from`; `show_undated` (default false) adds tasks without `due_date`. Sorted by `due_date` ascending, then `id` descending. Home: the agenda without a date range (all not-done tasks), or the saved filter `overview_filter_id` when set. | overdue task, default agenda → hidden; set overview filter → home shows its tasks |

## Out of scope

- Visual layout: table columns, card fields, month/week grids, drag preview, timeline bars.
- Per-view sharing or public view links.

## Decisions

- The source's Gantt timeline is specified as the `calendar` kind: same date-window rules, default window is the current month instead of −15/+55 days.
- Kanban default view is named `Board`; bucket titles are generic.
