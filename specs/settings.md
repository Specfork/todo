# Settings

**Status:** Draft · **Prefix:** SET

## Purpose

Where the data lives, who owns it, the per-user settings, and the local-time rules every other area uses.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| SET-1 | Server-based: the server stores all data and is the only source of truth. Clients read and write through its API and need a connection; there is no offline queue and no client-side merge. | server unreachable → write fails, nothing stored on the client |
| SET-2 | Every account owns one private data set: projects, views, buckets, tasks, labels, saved filters, reminders, notifications, settings. Every record carries `owner_id`; there is no sharing. Any read or write of another owner's record → `not_found`, same as a missing id. | user B reads user A's task id → `not_found` |
| SET-3 | Settings and defaults: `default_project_id`, `default_due_time` (`HH:MM`, default `09:00`), `quick_add_mode` (`symbols`\|`alt`\|`off`, default `symbols`), `quick_add_reminders` (list of signed second offsets, ≤ 5, default empty), `overview_filter_id?`. `time_zone` and `week_start` are the user's (`@specfork/auth:AUTH-1`). | new user → `default_due_time` `09:00` |
| SET-4 | Validation: `HH:MM` also accepts `H:MM`, range 00:00–23:59, else `invalid`; `overview_filter_id` must be an own saved filter, else `invalid`. | `default_due_time: 24:00` → `invalid` |
| SET-5 | Instants (task dates, reminders, `done_at`, `created`, `updated`) are stored in UTC and returned as RFC 3339 with `Z`. Local-time rules (date-only input, "today", date math, digest time, day steps) use the user's `time_zone` (`@specfork/auth:AUTH-1`) at the moment they are evaluated. Changing `time_zone` never rewrites stored instants; only later local-time evaluations change. | due `2026-05-04T07:00:00Z`, zone Berlin → shown 09:00 local; due 09:00 Berlin, switch zone to Lisbon → same instant, shown 08:00 |
| SET-6 | A local wall-clock time that does not exist (DST gap) resolves to the first valid minute after it; one that occurs twice (DST overlap) resolves to the first occurrence. | Berlin 2026-03-29 02:30 → 03:00 CEST (`01:00Z`) |
| SET-7 | Date-only input `YYYY-MM-DD` for a task date means that local day at `default_due_time` in `time_zone` (SET-6 applies). | `2026-06-01`, Berlin, default 09:00 → `2026-06-01T07:00:00Z` |
| SET-8 | Day arithmetic: a duration that is a whole multiple of 86,400 s is applied as that many calendar days in `time_zone`, keeping the local wall-clock time (SET-6 applies); any other duration is added as exact seconds. Used by [REP](recurrence.md) and [REM](reminders.md). | 09:00 Berlin on 2026-03-28 + 86,400 s → 09:00 on 2026-03-29 (23 h later) |

## Out of scope

- Offline-first clients, sync between devices without the server, end-to-end encryption.
- Several people sharing one data set; teams; link sharing.
- Interface preferences (theme, date display, sound on completion): client-side only.

## Decisions

- Server-based, as the source; a personal app still gets accounts so one instance can host several isolated users.
- `default_due_time` defaults to 09:00; the source's client picked the next of 9/12/15/18/21 o'clock.
- Day steps keep wall-clock time (SET-8); the source added exact seconds, so daily items drifted an hour at DST changes.
- `time_zone` and `week_start` (default Monday; the source defaulted to Sunday) come from `@specfork/auth:AUTH-1`.
- Works in either `@specfork/auth:AUTH-18` mode: `single_owner` for one person, `multi` for several isolated users.
