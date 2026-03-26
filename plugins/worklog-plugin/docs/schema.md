# Worklog Data Format Specification

This document defines the on-disk data format for the Worklog task management system. It is intended as a reference for any tool that needs to read or write worklog data — CLI tools, web frontends, IDE extensions, or other integrations.

## Directory Structure

All worklog data lives in `.claude/worklog/` relative to the project root.

```
.claude/worklog/
  .index.json            # JSON index (query cache + ID counter)
  001-fix-login-bug.md   # Task file
  002-add-dark-mode.md   # Task file
  ...
```

### File Naming Convention

Task files follow the pattern `{id}-{slug}.md` where:

- **id**: Zero-padded to 3 digits (e.g., `001`, `012`). IDs above 999 use as many digits as needed (e.g., `1000`).
- **slug**: Derived from the task title — lowercased, non-alphanumeric characters replaced with hyphens, consecutive hyphens collapsed, truncated to 50 characters, trailing hyphens stripped.
- The slug is for human readability only. The authoritative title is in the file's YAML frontmatter.

## Index File (`.index.json`)

The index caches all task metadata for fast querying. It is the recommended entry point for any read operation that needs to inspect multiple tasks.

### Schema

```json
{
  "counter": 3,
  "tasks": {
    "1": {
      "title": "Build auth system",
      "status": "open",
      "priority": "high",
      "labels": ["auth"],
      "blocked-by": [],
      "parent": null,
      "file": "001-build-auth-system.md",
      "created": "2026-03-26T10:30:00Z",
      "updated": "2026-03-26T10:30:00Z"
    }
  }
}
```

### Field Reference

| Field     | Type    | Required | Description                                                    |
| --------- | ------- | -------- | -------------------------------------------------------------- |
| `counter` | integer | Yes      | Last assigned task ID. Monotonically increasing, never reused. |
| `tasks`   | object  | Yes      | Map of task ID (string) to task metadata.                      |

#### Task Metadata Fields

| Field        | Type              | Required | Default    | Description                                                 |
| ------------ | ----------------- | -------- | ---------- | ----------------------------------------------------------- |
| `title`      | string            | Yes      | —          | Human-readable task title.                                  |
| `status`     | enum              | Yes      | `"open"`   | One of: `"open"`, `"closed"`.                               |
| `priority`   | enum              | Yes      | `"medium"` | One of: `"low"`, `"medium"`, `"high"`, `"critical"`.        |
| `labels`     | array of strings  | No       | `[]`       | Freeform labels for categorization.                         |
| `blocked-by` | array of integers | No       | `[]`       | Task IDs that must be closed before this task is ready.     |
| `parent`     | integer or null   | No       | `null`     | Parent task ID. `null` for top-level tasks.                 |
| `file`       | string            | Yes      | —          | Filename of the corresponding `.md` file (not a full path). |
| `created`    | string (ISO 8601) | Yes      | —          | Timestamp of task creation.                                 |
| `updated`    | string (ISO 8601) | Yes      | —          | Timestamp of last modification.                             |

### Priority Ordering

From highest to lowest: `critical` > `high` > `medium` > `low`.

### Status Values

| Status   | Meaning                                   |
| -------- | ----------------------------------------- |
| `open`   | Task is active (may be ready or blocked). |
| `closed` | Task is complete.                         |

## Task Markdown File Format

Each task is a markdown file with YAML frontmatter and a markdown body.

### Frontmatter

```yaml
---
id: 1
title: "Build auth system"
status: open
priority: high
labels: [auth, backend]
blocked-by: []
parent: null
created: 2026-03-26T10:30:00Z
updated: 2026-03-26T10:30:00Z
---
```

The frontmatter fields mirror the index metadata fields, with the addition of `id` (the integer task ID). The frontmatter is the source of truth — the index is a derived cache.

| Field        | Type              | Required | Notes                                                                           |
| ------------ | ----------------- | -------- | ------------------------------------------------------------------------------- |
| `id`         | integer           | Yes      | Unique task identifier. Present in `.md` but not in index (index uses the key). |
| `title`      | string            | Yes      | Always quoted in YAML to handle special characters.                             |
| `status`     | enum              | Yes      | `open` or `closed`.                                                             |
| `priority`   | enum              | Yes      | `low`, `medium`, `high`, or `critical`.                                         |
| `labels`     | array of strings  | No       | May be omitted if empty.                                                        |
| `blocked-by` | array of integers | No       | May be omitted if empty.                                                        |
| `parent`     | integer or null   | No       | May be omitted for top-level tasks.                                             |
| `created`    | string (ISO 8601) | Yes      | Set once at creation, never modified.                                           |
| `updated`    | string (ISO 8601) | Yes      | Updated on every modification.                                                  |

### Body Structure

The markdown body consists of two optional sections:

```markdown
## Description

Free-form markdown describing the task. May include any valid markdown:
lists, code blocks, links, images, etc.

## Comments

### 2026-03-26T14:00:00Z

First comment body — free-form markdown.

### 2026-03-26T15:30:00Z

Second comment body.
```

- **`## Description`**: The main task description. Always the first section if present.
- **`## Comments`**: A log of timestamped comments. Each comment is an `### {ISO-8601 timestamp}` heading followed by markdown content. Comments are appended in chronological order and should not be reordered or edited after creation.

## Dependency Model

### Explicit Dependencies (`blocked-by`)

A task may declare explicit dependencies on other tasks via the `blocked-by` field, which contains an array of task IDs. The task cannot be worked on until all referenced tasks have `status: "closed"`.

### Parent-Subtask Hierarchy (`parent`)

Tasks support a two-level hierarchy:

- **Top-level tasks** have `parent: null` (or omit the field).
- **Subtasks** reference their parent via `parent: N` where `N` is a top-level task ID.
- **Nesting limit**: Subtasks cannot have subtasks. A task whose `parent` references another subtask is invalid.

#### Implicit Blocking

A parent task is **implicitly blocked** by all of its open subtasks. This does not need to be (and should not be) expressed in `blocked-by`. The implicit blocking means:

- A parent cannot be worked on until all its subtasks are closed.
- Closing the last open subtask makes the parent "ready" — but the parent still requires explicit closure (it does not auto-close).

#### Cross-Parent Dependencies

A subtask may have `blocked-by` references to subtasks of a different parent. This creates cross-parent dependencies:

```
Task #1 (parent)
  #2 (subtask of #1)
  #3 (subtask of #1, blocked-by: [#2])

Task #4 (parent)
  #5 (subtask of #4, blocked-by: [#3])  ← cross-parent dependency
```

In this example, task #5 cannot start until task #3 (belonging to a different parent) is closed.

### Task Readiness

A task is considered **ready** (available to be worked on) when:

#### For subtasks and standalone tasks:

1. Status is `open`
2. All tasks in `blocked-by` have status `closed`

#### For parent tasks:

1. Status is `open`
2. All subtasks have status `closed` (implicit blocking)
3. All tasks in explicit `blocked-by` have status `closed`

### Work Order (Agent Ordering)

When an agent needs to determine which task to work on next:

1. Read `.index.json` to get all task metadata
2. Build the full dependency graph (explicit `blocked-by` + implicit parent→subtask relationships)
3. Identify all **ready** tasks (subtasks and standalone tasks first; parents only when all subtasks are done)
4. Sort ready tasks by priority (`critical` > `high` > `medium` > `low`), then by ID ascending as tiebreaker
5. The first task in the sorted list is the next task to work on

## Consistency Rules

### Source of Truth

The **markdown files** are the source of truth. The **`.index.json`** is a derived cache for performance. If they diverge, the markdown files win.

### Rebuilding the Index

To rebuild `.index.json` from scratch:

1. Scan `.claude/worklog/*.md` for all task files
2. Parse YAML frontmatter from each file
3. Extract the highest `id` value as the `counter`
4. Populate the `tasks` object from each file's frontmatter
5. Write the reconstructed `.index.json`

### Write Operations

Every operation that modifies task data must update **both** the `.md` file and `.index.json` atomically (within the same operation). The `updated` timestamp must be set to the current time on any modification.

### ID Assignment

- IDs are positive integers, starting at 1.
- IDs are monotonically increasing — the next ID is always `counter + 1`.
- IDs are never reused, even after a task is closed or deleted.
- The `counter` field in `.index.json` tracks the last assigned ID.

### Slug Generation Algorithm

Given a title string:

1. Convert to lowercase
2. Replace all non-alphanumeric characters with hyphens (`-`)
3. Collapse consecutive hyphens into a single hyphen
4. Truncate to 50 characters
5. Strip trailing hyphens

Example: `"Fix Login Bug on Mobile!"` → `fix-login-bug-on-mobile`
