---
name: worklog
description: "Local task management stored as markdown files in .claude/worklog/. Supports creating, listing, viewing, updating, closing, reopening, commenting on, and breaking down tasks with GitHub Issues-like workflow including parent/subtask hierarchy and dependency ordering. Triggers on: 'create task', 'add task', 'new task', 'list tasks', 'show tasks', 'view task', 'close task', 'reopen task', 'update task', 'comment on task', 'next task', 'what should I work on next', 'break down task', '/worklog', or any reference to managing local project tasks or worklog items."
---

# Worklog — Local Task Management

You are a task management system. Tasks are stored as markdown files in `.claude/worklog/` with a JSON index for fast querying. Follow these instructions precisely for each operation.

## Storage

- **Directory**: `.claude/worklog/` (relative to project root)
- **Index**: `.claude/worklog/.index.json` — caches all task metadata and the ID counter
- **Task files**: `.claude/worklog/{id}-{slug}.md` — human-readable task files with YAML frontmatter
- **Schema reference**: Read `plugins/worklog-plugin/docs/schema.md` for the complete data format specification

## Reading the Index

Before any operation, read `.claude/worklog/.index.json` using the Read tool. This single file contains all task metadata needed for listing, filtering, dependency resolution, and ordering.

If `.index.json` does not exist or is corrupted:

1. Glob `.claude/worklog/*.md` to find all task files
2. Read frontmatter from each file
3. Reconstruct the index (set `counter` to the highest `id` found)
4. Write the rebuilt `.index.json`

If the `.claude/worklog/` directory does not exist and the operation is a read (list, view, next), report "No worklog found. Create a task to get started." and stop.

## Operations

Parse the user's request to determine which operation to perform. If ambiguous, ask the user to clarify. If required parameters are missing, ask the user.

---

### Create

**Triggers**: "create task", "add task", "new task"

**Required**: title (ask if not provided)
**Optional**: description, priority (default: medium), labels (default: []), blocked-by (default: []), parent (default: null)

**Steps**:

1. Ensure `.claude/worklog/` directory exists (create if needed)
2. Read `.index.json` (or initialize with `{"counter": 0, "tasks": {}}` if missing)
3. Increment `counter` to get the new task ID
4. Generate the slug from the title:
   - Lowercase the title
   - Replace non-alphanumeric characters with hyphens
   - Collapse consecutive hyphens
   - Truncate to 50 characters
   - Strip trailing hyphens
5. **Validate** (if applicable):
   - If `parent` is set: verify the parent task exists and is a top-level task (its own `parent` is null). Reject if the parent is itself a subtask.
   - If `blocked-by` is set: verify all referenced task IDs exist. Warn about any that don't.
6. Create the `.md` file with frontmatter and body:

   ```yaml
   ---
   id: { id }
   title: "{title}"
   status: open
   priority: { priority }
   labels: [{ labels }]
   blocked-by: [{ blocked-by }]
   parent: { parent or null }
   created: { now ISO-8601 }
   updated: { now ISO-8601 }
   ---
   ## Description

   { description or "TODO: Add description" }
   ```

7. Add the task to `.index.json` (under the string key of the ID) and update `counter`
8. Write both files
9. Report: "Created task #{id}: {title}" and show its status (ready/blocked)

---

### List

**Triggers**: "list tasks", "show tasks", "/worklog"

**Optional filters**: status (default: open), priority, label, parent

**Steps**:

1. Read `.index.json`
2. Apply filters (default: show only open tasks)
3. Compute ready/blocked status for each task:
   - A task is **blocked** if any `blocked-by` reference is still open
   - A parent is **blocked** if any subtask is still open
   - Otherwise the task is **ready**
4. Display in hierarchical format — group subtasks under their parents:

```
Worklog: 5 open tasks (3 ready, 2 blocked)

  #1  [HIGH] Build auth system                    blocked (3 subtasks open)
    #2  [HIGH] Design API schema                   ready
    #3  [HIGH] Implement login endpoint             blocked by #2
    #4  [MED]  Add JWT validation                   blocked by #3

  #5  [MED]  Fix CSS bug on mobile                 ready
```

5. Standalone tasks (no parent) appear at the top level alongside parents
6. Sort by priority within each level, then by ID

---

### View

**Triggers**: "view task #N", "show task #N"

**Required**: task ID

**Steps**:

1. Read `.index.json` to get the task metadata and filename
2. Read the `.md` file for the full body (description + comments)
3. Display the task with:
   - All frontmatter fields formatted nicely
   - Ready/blocked status with details (which blockers are open/closed)
   - For parents: list all subtasks with their status
   - For subtasks: show parent title and link
   - The full description and comments

---

### Update

**Triggers**: "update task #N"

**Required**: task ID and at least one field to change
**Updatable fields**: title, priority, labels, blocked-by, parent

**Steps**:

1. Read `.index.json` and find the task
2. **Validate changes**:
   - If changing `blocked-by`: verify all referenced IDs exist; check for circular dependencies
   - If changing `parent`: verify target is a top-level task; verify this task has no subtasks of its own
   - If changing `title`: generate a new slug and rename the `.md` file
3. Apply changes to frontmatter in the `.md` file
4. Update `updated` timestamp
5. Update the task entry in `.index.json` (including `file` if renamed)
6. Write both files
7. Report what changed

**Circular dependency detection**: Before applying `blocked-by` changes, walk the dependency graph from each new blocker. If any path leads back to the current task, reject the change and explain the cycle.

---

### Close

**Triggers**: "close task #N", "complete task #N", "done with task #N"

**Required**: task ID

**Steps**:

1. Read `.index.json` and find the task
2. Set `status: closed` in the `.md` frontmatter
3. Update `updated` timestamp
4. Update `.index.json`
5. Write both files
6. Check what this unblocks:
   - Find all open tasks that had this task in their `blocked-by` — if this was their last open blocker, report them as now **ready**
   - If this was the last open subtask of a parent, report the parent as now **ready**
7. Report: "Closed task #{id}: {title}" followed by any newly unblocked tasks

---

### Reopen

**Triggers**: "reopen task #N"

**Required**: task ID

**Steps**:

1. Read `.index.json` and find the task
2. Set `status: open` in the `.md` frontmatter
3. Update `updated` timestamp
4. Update `.index.json`
5. Write both files
6. Check what this re-blocks:
   - Find all open tasks that have this task in their `blocked-by` — they are now blocked again
   - If this is a subtask, its parent is now blocked again
7. Report: "Reopened task #{id}: {title}" followed by any tasks that are now re-blocked

---

### Comment

**Triggers**: "comment on task #N", "add comment to task #N", "note on task #N"

**Required**: task ID, comment text

**Steps**:

1. Read `.index.json` to find the task's filename
2. Read the `.md` file
3. If no `## Comments` section exists, append one after the description
4. Append the new comment:

   ```markdown
   ### {now ISO-8601}

   {comment text}
   ```

5. Update `updated` timestamp in frontmatter
6. Update `.index.json` (updated timestamp only)
7. Write both files
8. Report: "Added comment to task #{id}"

---

### Next

**Triggers**: "next task", "what should I work on next", "what's next"

**Steps**:

1. Read `.index.json`
2. Build the dependency graph (explicit `blocked-by` + implicit parent→subtask)
3. Find all **ready** tasks:
   - Open tasks that are NOT parents, where all `blocked-by` references are closed
   - Parent tasks where all subtasks are closed AND all `blocked-by` references are closed
4. Prioritize: subtasks and standalone tasks first, then parents
5. Sort by priority (critical > high > medium > low), then by ID ascending
6. Display the top ready task with its full context:
   - If it's a subtask, show the parent's description for context
   - Show the task's description
   - Show any related blocked tasks that will be unblocked when this is completed
7. If no ready tasks exist but open tasks remain, report the blockage and explain what's preventing progress

---

### Breakdown

**Triggers**: "break down task #N", "split task #N", "decompose task #N"

**Required**: task ID

**Steps**:

1. Read `.index.json` and find the task
2. Verify the task is a top-level task (not already a subtask)
3. Verify the task has no existing subtasks (or confirm with user if it does — new subtasks will be added)
4. Ask the user for subtask details — titles, descriptions, dependencies between them
5. Create each subtask using the **Create** operation with `parent` set to this task's ID
6. If the user specifies ordering between subtasks, set appropriate `blocked-by` references
7. Report the breakdown summary showing the parent and all new subtasks with their dependency chain

---

## Important Rules

### Timestamps

- Always use ISO 8601 format: `YYYY-MM-DDTHH:MM:SSZ`
- Use UTC timezone

### Consistency

- Every write operation must update BOTH the `.md` file and `.index.json`
- Always update the `updated` timestamp on any modification
- The `.md` frontmatter is the source of truth; `.index.json` is the cache

### Hierarchy

- Two-level maximum: top-level tasks and subtasks. No deeper nesting.
- A parent is implicitly blocked by all open subtasks — never add subtasks to `blocked-by`
- Parents become ready when all subtasks close, but require explicit closure

### IDs

- Monotonically increasing positive integers, starting at 1
- Never reused, even after task closure
- Zero-padded to 3 digits in filenames (or more digits if ID > 999)

### YAML Frontmatter

- Always quote the `title` string to handle special characters
- Use flow syntax for arrays: `[item1, item2]`
- Use `null` (not empty string) for absent `parent`
