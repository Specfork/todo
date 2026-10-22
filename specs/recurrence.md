# Recurrence

**Status:** Draft · **Prefix:** REP

## Purpose

Repeating tasks: one task that reopens with new dates each time it is completed.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| REP-1 | Fields: `repeat_mode`: `interval` (default) \| `monthly` \| `from_completion`; `repeat_after`: seconds, 0–315,360,000 (10 years), default 0; `repeat_months`: 1–120, default 1, used only by `monthly`. Out of range → `invalid`. | `repeat_after: -5` → `invalid` |
| REP-2 | A task repeats when `repeat_mode` is `monthly`, or when it is `interval`/`from_completion` with `repeat_after` > 0. Completing a repeating task (by done flag or done bucket) keeps `done` false, sets `done_at` to now, moves dates and reminders per REP-3..6, unticks every ticked checkbox in `description`, and puts the task in each kanban view's default bucket. | daily task completed → `done` false, due moved |
| REP-3 | `interval`: each set date (`due_date`, `start_date`, `end_date`) and each absolute reminder moves on its own by the smallest whole number k ≥ 1 of `repeat_after` steps that puts it strictly after now. Unset dates stay unset. Steps follow [SET-8](settings.md). | daily, due 3 days ago 09:00, now 10:00 → due tomorrow 09:00 |
| REP-4 | `monthly`: each set date and absolute reminder moves forward `repeat_months` calendar months in `time_zone`, once, keeping local time; a day past the month's end becomes its last day. If start and end are both set, end = new start + old (end − start). | due Jan 31 09:00, 1 month → Feb 28 09:00 (2027) |
| REP-5 | `from_completion`: new `due_date` = now + `repeat_after` (SET-8); `start_date` and `end_date` keep their old offsets from `due_date`. Without a due date: start = now + step and end keeps its offset from start; with only an end date, end = now + step. | weekly, completed Wed 18:00 → due next Wed 18:00 |
| REP-6 | `from_completion` reminders: the earliest absolute reminder becomes now + `repeat_after`; the others keep their offsets from it. | reminders 09:00 and 10:00 → new pair 1 h apart |
| REP-7 | After any move, relative reminders are recomputed from the new dates ([REM-3](reminders.md)) and every reminder's `sent_at` is cleared. | relative −1 h to due, due moved a day → reminder moved a day, unsent |
| REP-8 | Setting `repeat_after` to 0 in `interval` or `from_completion` mode stops repeating; the next completion is final. | repeat removed, complete → `done` true |

## Out of scope

- Rule-based repeats (every 2nd Tuesday, weekdays only), end dates or counts for a series, per-occurrence history.

## Decisions

- `repeat_months` added: the source's monthly mode was fixed at one month, and its quick-add turned other month and year counts into no repeat.
- `interval` and `monthly` keep their source meaning: `interval` catches up past now, `monthly` moves once even if still in the past.
