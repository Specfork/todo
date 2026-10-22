# Subtasks

**Status:** Draft · **Prefix:** REL

## Purpose

Links between tasks: parent and subtask, plus related, duplicate, blocking, order and copy links.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| REL-1 | Relation `{task_id, other_task_id, kind}`. Kinds and their inverses: `subtask` ↔ `parent`, `related` ↔ `related`, `duplicate_of` ↔ `duplicates`, `blocking` ↔ `blocked_by`, `precedes` ↔ `follows`, `copied_from` ↔ `copied_to`. Other kinds → `invalid`. | kind `child` → `invalid` |
| REL-2 | Creating a relation also stores its inverse; deleting either side deletes both. | A `subtask` B → B shows A as `parent` |
| REL-3 | A task related to itself → `invalid`. The same `(task_id, other_task_id, kind)` twice, or its inverse, → `conflict`. One pair may hold several different kinds. | A `subtask` B exists, add B `parent` A → `conflict` |
| REL-4 | `subtask`/`parent` links must not form a cycle of any length → `cycle`. Other kinds may form cycles. | A ⊃ B ⊃ C, make A a subtask of C → `cycle` |
| REL-5 | Both tasks must belong to the owner; they may be in different projects. A trashed task's relations are hidden and come back on restore; purge deletes them. | link tasks in Inbox and Work → allowed |
| REL-6 | A task read includes `related_tasks` grouped by kind, each `{id, identifier, title, done, project_id}`. | read A → `related_tasks.subtask` lists B |
| REL-7 | Listing with `nest_subtasks=true` returns only tasks without a `parent` in the result, each with its matching subtasks nested at any depth; nested tasks do not count toward `per_page`. | A ⊃ B, nested list → one top item A containing B |

## Out of scope

- Dependency scheduling (blocked tasks cannot be completed, auto-moving followers).
- Checklists as records; description checkboxes are plain rich text ([REP-2](recurrence.md) clears them).

## Decisions

- No automatic roll-up: completing a parent or all subtasks changes nothing else ([TASK-8](tasks.md)), as in the source.
