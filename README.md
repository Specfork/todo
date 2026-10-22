---
specfork: 1
name: "@specfork/todo"
summary: Personal task manager with projects, subtasks, dates, priorities, labels, repeats, reminders, list/kanban/calendar views, filters, quick add, import and export.
license: CC-BY-4.0
requires:
  "@specfork/auth": "^0.3.0"
  "@specfork/notifications": "^0.1.0"
  "@specfork/privacy": "^0.1.0"
derived_from:
  repo: https://github.com/go-vikunja/vikunja
  commit: bc13d2095eccfe1d42904bc5606ea3abbdb84526
  url: https://vikunja.io
  license: AGPL-3.0-or-later
---

# Todo

One person keeps their tasks in nested projects, plans them with dates, priorities, labels and
repeats, gets reminded on time, and looks at them as a list, a table, a kanban board, a calendar
or a cross-project agenda. Everything can be exported in one archive and imported again.

- **Server-based, single user per account.** The server is the source of truth; each account
  (`@specfork/auth`) owns one private data set. No teams, sharing or collaboration.
- **Export all my data** (`@specfork/privacy`) returns every project, view, bucket, task (trash
  included), label, reminder, relation, saved filter and setting ([DATA-1](specs/data.md)).
- **Reminders and the overdue digest** arrive through `@specfork/notifications` ([REM](specs/reminders.md)).

Behaviour derived from Vikunja's open-source code (AGPL-3.0-or-later), written fresh and stack-free.
Features Vikunja unlocks only with a paid licence key (admin panel, time tracking, audit logs, user
invites) were not used.
Not affiliated with or endorsed by Vikunja or its contributors. The name is used only to credit the source.

| Area | Prefix | Covers |
|---|---|---|
| [settings.md](specs/settings.md) | SET | Server model, data ownership, user settings, local time, day arithmetic |
| [projects.md](specs/projects.md) | PRJ | Projects, nesting, inbox, archive, delete, duplicate, favorites |
| [tasks.md](specs/tasks.md) | TASK | Task fields, priorities, dates, done and undo, trash, bulk, labels |
| [subtasks.md](specs/subtasks.md) | REL | Subtasks and other task relations |
| [recurrence.md](specs/recurrence.md) | REP | Repeat modes and how completing a repeating task moves its dates |
| [reminders.md](specs/reminders.md) | REM | Absolute and relative reminders, notification types, overdue digest |
| [views.md](specs/views.md) | VIEW | List, table, kanban and calendar views, buckets, positions, agenda |
| [query.md](specs/query.md) | QRY | Filter language, search, sorting, paging, saved filters, favorites |
| [quick-add.md](specs/quick-add.md) | QADD | Parsing one line of text into a task |
| [data.md](specs/data.md) | DATA | Export contents, import rules, CSV import, delete-all, inventory |

All instants are stored in UTC; API names are `snake_case`.
Errors are named codes: `invalid`, `not_found`, `taken`, `conflict`, `cycle`, `archived`,
`bucket_full`, `limit_reached`, `busy`, `gone`, `filter_too_complex`.
