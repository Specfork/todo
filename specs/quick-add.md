# Quick add

**Status:** Draft · **Prefix:** QADD

## Purpose

Turn one line of text into a task with labels, project, priority, due date and repeat.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| QADD-1 | `quick_add {text, project_id?}` parses `text` by `quick_add_mode`: `symbols` → `*label`, `+project`, `!priority`; `alt` → `@label`, `#project`, `!priority`; `off` → no parsing. A token runs to the next space, or is a quoted value right after the prefix. Matched tokens are removed and spaces collapsed. | `Buy milk *shop !3` → title `Buy milk`, label `shop`, priority 3 |
| QADD-2 | Priority tokens count only for 0–5; others stay in the title. Only the first project token counts, matched case-insensitively by title, then by `identifier`; no match → the token stays in the title. The task goes to the matched project, else `project_id`, else `default_project_id`. | `+nowhere x` → title `+nowhere x` in Inbox |
| QADD-3 | Labels match own labels by title, ignoring case; unknown ones are created with a random `hex_color`; repeated label tokens count once. | `*new *new` → one new label |
| QADD-4 | Date words: `today`, `tonight` (21:00), `tomorrow`, `next week` (+7 days), `next month` (1st of next month), `end of month`, `this weekend` (coming Saturday, today if Saturday), weekday names full or 3-letter with optional `next` (next such day after today), `in N hours\|days\|weeks\|months`, numeric `YYYY-MM-DD`, `YYYY/MM/DD`, `M/D[/YYYY]`, `D.M[.YYYY]`, `<month name> D` or `D <month name>`, optionally followed by `at H[:MM][am\|pm]`. Days are in `time_zone`. | `call mom fri at 6pm` on Mon → due Fri 18:00 local |
| QADD-5 | The first date phrase becomes `due_date` and is removed; without a time it takes `default_due_time`; a date without a year that has passed this year goes to next year. | `Dec 1` on Dec 5 → next year Dec 1 09:00 |
| QADD-6 | Repeat phrases: `every [N] hour(s)\|day(s)\|week(s)\|month(s)\|year(s)` (N as digits or one…ten), `hourly`, `daily`, `weekly`, `monthly`, `yearly`. Hours, days and weeks set `interval` mode; months set `monthly` with `repeat_months` N; years set `monthly` with 12 × N. No date given → due today at `default_due_time`. | `water plants every 3 days` → `repeat_after` 259,200, due today |
| QADD-7 | Text wrapped wholly in quotes is taken literally without the quotes; text that is only tokens is taken literally as the title. A created task with `due_date` and no reminders gets one relative reminder per `quick_add_reminders` offset ([REM-3](reminders.md)). | `"*literal"` → title `*literal` |

## Out of scope

- Assigning people; natural-language parsing in other languages.

## Decisions

- Mode names are generic (`symbols`, `alt`); the source named its second mode after another product.
- Parsing may run on client or server, but results must match these rules.
