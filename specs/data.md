# Data

**Status:** Draft · **Prefix:** DATA

## Purpose

What todo's export holds, its import and CSV import rules, and what delete-all and the inventory cover.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| DATA-1 | Export (`@specfork/privacy:PRV-1`), `format` `@specfork/todo`, `version` 1, one JSON Lines file per entity: `settings`, `projects` (archived too), `views`, `buckets`, `tasks` (trashed too, with `deleted_at`; `label_ids`, per-view `positions` and `bucket_ids`), `labels`, `reminders`, `relations`, `saved_filters`. Every field of this spec; instants UTC. Notifications are not exported. | export → every task id once in `tasks.jsonl` |
| DATA-2 | Native import (`@specfork/privacy:PRV-5`): an export ZIP of ≤ 256 MB and ≤ 100,000 items (projects, views, buckets, tasks, labels, reminders, relations, saved filters), else `invalid` before anything is written. Runs in the background, one at a time per user (`busy`); ends with `import_finished` or `import_failed` ([REM-8](reminders.md)). | 100,001 tasks → `invalid`, no data added |
| DATA-3 | Import fixes conflicts instead of failing: a label with the same title and color as an existing own label is reused; a clashing project `identifier` is cleared; tasks with empty titles are skipped; relations to missing tasks are dropped; archived projects are archived after their tasks are created. | import twice → projects doubled, labels not; import `HOME` when `HOME` exists → identifier empty |
| DATA-4 | CSV import, step 1: a UTF-8 file of ≤ 100,000 rows with a header row; the delimiter (`,` `;` tab `\|`) is detected; the server returns headers, a suggested mapping and the first 5 rows. Step 2: the user maps each column to `title`, `description`, `due_date`, `start_date`, `end_date`, `done`, `priority`, `labels`, `project`, `reminder` or `ignore` and may preview the first 5 tasks. A mapping without `title` → `invalid`. All or nothing, like DATA-2. | `;`-separated file → delimiter `;` |
| DATA-5 | CSV values: dates read in the chosen format in `time_zone`; `done` true for `true`, `yes`, `1`, `done`, `completed` (any case); `priority` 0–5 clamped or the words low/medium/high/urgent → 1/2/3/4; `labels` comma-separated; `reminder` an absolute time. Everything goes under one new top-level project `Imported from CSV`, one child per distinct `project` value. | row `done=Yes` → task done |
| DATA-6 | Delete all my data (`@specfork/privacy:PRV-6`) and account deletion (`@specfork/privacy:PRV-7`) remove the whole [SET-2](settings.md) data set, trash included; a new empty `Inbox` becomes the default ([PRJ-6](projects.md)). | delete all → no projects but a new empty Inbox |
| DATA-7 | Inventory (`@specfork/privacy:PRV-9`): projects, views, tasks, labels, reminders, relations, saved filters and settings, kept until deleted; trashed tasks 30 days ([TASK-11](tasks.md)); notifications per `@specfork/notifications:NTFY-12`. No third parties. | manual: inventory lists each entity of DATA-1 |

## Out of scope

- Importers for other products' accounts or files; scheduled or continuous sync; whole-instance backup and restore.
- Attachments and backgrounds in the archive (not part of this spec).

## Decisions

- Packaging, delivery, limits and re-authentication come from `@specfork/privacy`; todo lists only its entities and import fixes. The source used a product-named version file.
- Imports are all or nothing; the source could stop part-way.
- CSV priority words map to this spec's scale; the source's CSV mapping disagreed with its own priority labels.
