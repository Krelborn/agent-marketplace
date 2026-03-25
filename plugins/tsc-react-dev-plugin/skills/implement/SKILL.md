---
name: implement
description: "Plan and implement code changes for TypeScript/React codebases. Orchestrates investigation and planning interactively in the main context, then forks execution (coder/reviewer loops, final review, verification) to a dedicated subagent. Triggers on: 'implement', 'plan and execute', 'make this change', 'build this feature', '/implement', or when the user describes a substantial code change to plan and execute."
---

# Implement Skill

You are a **coordination-only** orchestrator. You MUST NOT write or modify any code directly. Your job is to investigate, plan a change with the user, then hand off execution to a subagent.

## Input

The user has provided: $ARGUMENTS

If the argument is a file path, read the file for the change description. Otherwise treat the argument as an inline description of the change.

## Phase 1: Investigate

Use an Explore subagent to:

1. Understand the change being requested
2. Trace through the codebase to identify every file and module affected
3. Map dependencies — what depends on what, and in what order things need to change
4. Identify any risks, edge cases, or ambiguities

Return a concise summary of findings. Do NOT load full file contents into your own context — only the dependency map and key observations.

## Phase 2: Plan

Based on the investigation, produce a structured plan with these sections:

1. **Summary**: One paragraph describing the change and its blast radius
2. **Layers**: Break the work into ordered layers where each layer is a coherent unit of change:
   - Each layer should have a clear scope (e.g., "Update the schema", "Fix downstream types", "Update tests")
   - Note dependencies between layers (which must complete before others can start)
   - Identify any layers that can run in parallel
3. **Risks**: Anything that could go wrong
4. **Verification**: How to confirm the change is correct (type checker, test suite, manual checks)

### Testing requirements

Unless the user explicitly states otherwise, **100% test coverage is required** for all new or modified code. For each layer that involves tests, note in the plan:

- Which source files need test coverage
- Whether tests are new or modifications to existing tests

### Resolve ambiguities first

If the investigation surfaced any ambiguities, unknowns, or decisions that require user input, use `AskUserQuestion` to resolve **every one of them** before presenting the plan. Do not list unresolved questions in the plan — resolve them now.

### Present the plan and get approval

Output the complete plan **directly in your response** as formatted text. Never write the plan to a file during this phase. Then use `AskUserQuestion` to ask the user to approve the plan, request changes, or reject it.

**Approval loop — this is critical:**

1. After presenting the plan, ask the user to approve, request changes, or reject.
2. If the user responds with **anything other than clear, unambiguous approval** (e.g., they ask questions, suggest modifications, raise concerns, or request clarification), you MUST:
   - Answer their questions and/or revise the plan as needed
   - Re-present the updated plan (or confirm no changes were needed)
   - Ask for approval again using `AskUserQuestion`
3. **Repeat step 2** as many times as necessary. There is no limit on iterations.
4. Only treat responses like "approved", "go ahead", "looks good", "yes", "ship it", or similarly unambiguous affirmatives as approval.
5. **Do NOT proceed to Phase 3 until the user has given explicit, unambiguous approval.** A question or change request is NOT approval, even if it sounds positive. When in doubt, ask again.

## Phase 3: Dispatch Execution

After the user approves the plan:

### Persist the plan

Write the approved plan to a file so the execution subagent can read it:

- **If a plan mode file path is available** (check your system message for a plan file path): write the plan there.
- **Otherwise**: write the plan to `.claude/implement-plan.md` in the project root.

Record the path — you will pass it to the execution subagent.

### Launch the execution subagent

Dispatch a single Agent subagent (with NO `subagent_type` so it retains access to the Agent tool for dispatching its own coder/reviewer subagents). Include in the prompt:

1. The plan file path
2. Instruction: "Read `references/execution-playbook.md` relative to the implement skill for your full execution instructions. Follow them."
3. The skill directory path (so the subagent can resolve relative references)
4. Key file paths from the investigation phase
5. Any user-specified constraints or preferences

Example dispatch prompt:

```
You are an execution orchestrator. Your job is to implement an approved plan using Coder and Code Review subagents.

**Plan file**: [path]
**Skill directory**: [path to skills/implement/]

Read the execution playbook at [skill-dir]/references/execution-playbook.md for your full instructions. Follow them precisely.

Key files identified during investigation:
- [file list]

Constraints:
- [any user constraints]
```

After dispatching, report to the user that execution has been handed off and summarise what the subagent will do.

## Rules

- **Never write code yourself** — all implementation and review happens via subagents
- **Minimise your context** — pass subagents only what they need; summarise, don't copy
- **Log progress** — after each phase transition, briefly note what completed and what is dispatching next
- **Prefer small briefs** — focused subagent tasks over large monolithic ones
- **User checkpoints** — present the plan before executing; surface persistent issues rather than looping forever
- **Handle failures** — if a subagent fails or gets stuck after 3 attempts, report to the user and ask how to proceed
- **Plan in chat, not in files** — always output the plan directly in your response text during Phase 2; only write to file in Phase 3 for handoff
- **Resolve before executing** — use AskUserQuestion to resolve every ambiguity before starting Phase 3; never proceed with unresolved questions
- **Explicit approval required** — do not start execution until the user has given clear, unambiguous approval. If the user responds to an approval prompt with questions, concerns, or change requests, answer them, revise the plan if needed, and ask for approval again. Repeat until approval is given. A question is never approval.
