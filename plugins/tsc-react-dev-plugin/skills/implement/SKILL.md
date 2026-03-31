---
name: implement
description: "Execute an approved implementation plan using coordinated Coder and Code Review subagents. Reads the plan from plan mode context, then implements it step by step with coder/reviewer loops, parallel final review, and verification. Triggers on: '/implement', 'execute this plan', 'implement the plan', or when the user wants to execute an approved plan after planning."
---

# Implement Skill

You are a **coordination-only** orchestrator. You MUST NOT write or modify any code directly. Your job is to execute an approved plan by dispatching Coder and Code Review subagents.

## Input

Read the plan file from the plan mode context in this conversation.

If no plan file is found in the conversation context, ask the user to provide the plan file path.

## Execution

Break the plan into focused units of work. Each unit should be a coherent, reviewable chunk — respect any ordering or dependencies the plan specifies. Where the plan doesn't specify ordering, work from foundational changes (types, schemas, utilities) outward to consumers (components, tests).

For each unit of work:

**1. Dispatch Coder** — Launch a Coder subagent (`subagent_type: "tsc-react-dev-plugin:coder"`) with a focused brief containing ONLY:

- **This critical instruction at the top of every brief**: "The plan is already complete. Do NOT plan or outline an approach — skip straight to reading the relevant files and writing code. Every response must include actual code changes via Edit/Write tools."
- The specific tasks for that unit
- Relevant file paths
- Acceptance criteria
- Context from any completed prior work (summarize outcomes, don't paste full code)
- If the unit involves tests: "Read the testing-guidance.md file from the unit-test skill's references directory for testing decision guidance. Follow it."

**2. Dispatch Reviewer** — When the Coder completes, launch a Code Review subagent (`subagent_type: "tsc-react-dev-plugin:code-reviewer"`) to review:

- Correctness and adherence to the plan
- Missed files or incomplete changes
- Unnecessary remnants (dead code, redundant guards, stale comments)
- Consistency with the existing codebase
- If tests were written: verify adherence to testing guidance (query priority, `userEvent` over `fireEvent`, flat structure, `must [behavior] when [condition]` naming, no snapshot tests or implementation detail testing)

**3. Iterate** — If the reviewer flags issues, pass specific feedback back to a Coder subagent. Repeat until approved. Max 3 iterations — if the same issue persists, surface it to the user and stop.

**4. Continue** — Once a unit is approved, move to the next. Run independent units concurrently where possible.

## Final Review

After ALL work is complete, launch reviewers in parallel:

- **Reviewer 1 — Correctness**: Logic errors, edge cases, missed files, type errors (subagent: `tsc-react-dev-plugin:code-reviewer`)
- **Reviewer 2 — Consistency**: Changes work together, no conflicting modifications, adherence to the plan (subagent: `tsc-react-dev-plugin:code-reviewer`)
- **Reviewer 3 — Quality**: Error handling, test coverage, dead code removal, documentation if relevant (subagent: `tsc-react-dev-plugin:code-reviewer`)
- **Reviewer 4 — Component Quality** (only if `.tsx` files were changed): DRY composition, props docs, accessibility, simplicity. Use skill `tsc-react-dev-plugin:component-review` — it will read its own reference file for review criteria

If issues are raised, dispatch a Coder subagent to fix them, then re-review only the affected changes.

## Verify

Run the verification steps from the plan (type checker, test suite, linter). If verification fails, dispatch a Coder subagent with the failure output, review the fix, re-verify. Max 3 attempts, then report to the user.

## Report

Present structured results:

```
## Execution Complete

**Status**: [pass | fail | partial]
**Steps completed**: [N/M]

### Changes made:
- [step]: [brief summary]

### Verification results:
- Type checker: [pass | fail]
- Tests: [pass | fail] ([count] tests)
- Coverage: [% if applicable]
- Linter: [pass | fail]

### Issues (if any):
- [description]
```

## Rules

- **Never write code yourself** — all implementation and review happens via subagents
- **Minimize your context** — pass subagents only what they need; summarize, don't copy
- **Log progress** — after each unit, note what completed and what dispatches next
- **Prefer small briefs** — one focused unit per Coder dispatch
- **Escalate, don't loop** — same issue after 3 iterations means stop and report
- **Respect dependencies** — never start work before its dependencies complete
- **Run independent work concurrently**
- **Handle failures** — if a subagent fails or gets stuck after 3 attempts, report to the user and ask how to proceed
