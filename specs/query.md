# Query

**Status:** Draft · **Prefix:** QRY

## Purpose

How tasks are filtered, searched, sorted and paged, plus saved filters and the favorites list.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| QRY-1 | Filter: conditions `field op value` joined by `&&` and `\|\|`, `&&` binding tighter, parentheses for grouping. Ops: `=`, `!=`, `>`, `>=`, `<`, `<=`, `like`, `in`, `not in`. Values may be quoted with `'` or `"`; `in` lists are comma-separated. Unknown op or field → `invalid`. | `priority ~ 3` → `invalid` |
| QRY-2 | Fields: `id`, `index`, `title`, `description`, `done`, `done_at`, `due_date`, `start_date`, `end_date`, `priority`, `percent_done`, `hex_color`, `repeat_after`, `created`, `updated`, `project` (id), `labels` (id), `reminders`, `bucket_id`. | `labels in 3, 7` → tasks with label 3 or 7 |
| QRY-3 | `labels` and `reminders` match when any item matches; with `!=` or `not in`, when no item matches. `like` is a case-insensitive substring match. | `labels != 3` → tasks without label 3 |
| QRY-4 | Date values: RFC 3339; `YYYY-MM-DD` (local midnight); `YYYY-MM-DD HH:MM` (local); or date math `now` followed by any of `+N`/`-N` with unit `s m h d w M y` and rounding `/d /w /M /y` to the start of the local unit (weeks start on `week_start`). Other text → `invalid`. Local means `filter_time_zone` when the request gives one (unknown → `invalid`), else the user's `time_zone`. | `due_date < now/d+1d` → due before tomorrow 00:00 local; zone Tokyo, `now/d` → Tokyo midnight |
| QRY-5 | `include_nulls=true`: every condition also matches tasks whose field is empty (null, 0 for numbers, no labels or reminders). | `due_date < now`, include nulls → undated tasks too |
| QRY-6 | A filter over 16 KB or nested deeper than 100 parentheses → `filter_too_complex`; unbalanced parentheses → `invalid`. | 101 nested groups → `filter_too_complex` |
| QRY-7 | Search `s`: trimmed; case-insensitive substring of `title` or `description`; `#N` also matches tasks with `index` N. Combined with the filter by AND. | `s=#12` → task with index 12 |
| QRY-8 | Sort: `sort_by` (one or more of the QRY-2 scalar fields, or `position` within a view) with `order_by` `asc` (default) \| `desc`; empty values last; `id` ascending breaks ties. Default: `position` in a view, else `id`. | sort `due_date` → undated tasks last |
| QRY-9 | Paging: `page` from 1, `per_page` default 50, maximum 50 (larger values capped), below 1 → `invalid`; responses carry `total` and `total_pages`. | `per_page=500` → 50 items |
| QRY-10 | Saved filter: `{id, title (1–250 chars), description?, filter, sort?, include_nulls (default false), is_favorite, created, updated}`; the filter is checked on save. Its tasks come from all own non-archived projects. | save `done = false && priority >= 4` → urgent open tasks |
| QRY-11 | Saved filters are listed with projects as read-only pseudo-projects (`kind: saved_filter`) with the VIEW-2 views, the list view without the `done = false` filter. Creating a task in one → `invalid`. In its manual kanban views, tasks that newly match are added to the default bucket (date-based filters re-checked at least every minute). | create task in saved filter → `invalid` |
| QRY-12 | Favorites pseudo-project (`kind: favorites`): listed only while at least one non-trashed task is favorite; holds those tasks; has list (filter `done = false`), table and calendar views and no kanban; read-only. | unfavorite last task → pseudo-project gone |

## Out of scope

- Full-text ranking or fuzzy search; filter autocomplete; searching comments or attachments.

## Decisions

- `&&` binds tighter than `\|\|`; the source applied joins left to right.
- Date math defaults to the user's zone everywhere; the source used the server zone outside saved filters.
- Assignee and creator fields are gone: one user owns everything.
