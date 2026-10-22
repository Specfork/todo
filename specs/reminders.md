# Reminders

**Status:** Draft · **Prefix:** REM

## Purpose

Reminders on tasks, when they fire, the notification types todo sends, and the daily overdue digest.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| REM-1 | Reminder: `{id, task_id, remind_at? (UTC instant), relative_to?: due_date\|start_date\|end_date, relative_period (signed seconds, default 0), sent_at?}`; at most 20 per task, else `invalid`. | 21st reminder → `invalid` |
| REM-2 | Absolute reminder: `relative_to` null and `remind_at` given. A non-zero `relative_period` without `relative_to` → `invalid`. | `{relative_period: -3600}` → `invalid` |
| REM-3 | Relative reminder: `remind_at` = the `relative_to` date + `relative_period` (negative = before), using [SET-8](settings.md); recomputed on every task save. If that date is unset, `remind_at` is null and the reminder never fires but is kept. | due 10:00Z, −3,600 → 09:00Z; clear due → `remind_at` null |
| REM-4 | Reminders with the same `remind_at` (to the second) on one task merge into one; reads return them by `remind_at` ascending. Any change of `remind_at` clears `sent_at`. | two at 09:00 → one stored |
| REM-5 | A scheduler runs at least once a minute. A reminder fires when `remind_at` ≤ now, `sent_at` is null, the task is not done, not trashed and not in an archived project. Firing sets `sent_at`, so each reminder fires once. | done task, reminder time passes → nothing sent |
| REM-6 | Missed runs: a reminder found up to 24 h late is still delivered, marked `late`; one found more than 24 h late gets `sent_at` without delivery. | server down 2 h → reminder delivered on restart |
| REM-7 | `remind_at` is an absolute instant: it fires at the same moment whatever `time_zone` says, and changing `time_zone` does not move it ([SET-5](settings.md)). Messages show times in the user's current `time_zone`. | reminder 07:00Z, zone Berlin → fires 07:00Z, text says 09:00 |
| REM-8 | Notification types (`@specfork/notifications:NTFY-1`), recipient always the owner: `task_reminder` (subject: task; in-app `required`, email default `off`), `overdue_digest` (subject: the local date; in-app on, email default `instant`), `import_finished` and `import_failed` (subject: the import; in-app on, email `off`). | user turns `task_reminder` email on → reminders also mailed |
| REM-9 | A firing reminder creates or bumps (`@specfork/notifications:NTFY-2`) the task's `task_reminder`: title = task title, body = its due time in the user's `time_zone`, marked `late` under REM-6. Reminders of one task firing in the same run count once. Notifications of a trashed task are hidden until restore; purge deletes them (`@specfork/notifications:NTFY-13`). | 2 reminders same minute → 1 notification, count 1 |
| REM-10 | Overdue digest: once per local day at the user's `digest_time` (`@specfork/notifications:NTFY-8`; SET-6 applies), when tasks exist that are not done, not trashed, not in archived projects, with `due_date` before that moment, one `overdue_digest` is created; else nothing. Lines sorted by `due_date` ascending, each: title, identifier, project title, whole hours overdue; past the 2,000-char body limit the list ends at a whole line with `+N more`. A missed digest is created later the same local day; at most one per local date. | 2 overdue tasks at 08:00 → one digest with 2 lines; server down 08:00–11:00 → digest at 11:00 |

## Out of scope

- Channels and inbox features beyond `@specfork/notifications`.
- Daily summary of tasks due today (the digest lists overdue tasks only).

## Decisions

- At most 20 reminders per task; the source had no limit.
- `sent_at` with a 24 h catch-up (REM-5, REM-6); the source fired only inside the current minute and silently lost reminders while down.
- Archived projects send no reminders, matching the digest; the source still fired them.
- The digest uses the user's local date; the source used the server's date, so far-off zones could miss it.
- Inbox, read state, snooze, email delivery and preferences come from `@specfork/notifications`; todo only defines the types, when they fire and their text. The digest is one notification per day, so it lists every overdue task, not only new ones.
- Recipient is always the owner: no assignees or subscribers in a personal app.
