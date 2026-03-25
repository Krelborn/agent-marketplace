---
name: unit-test
description: "Plan and execute unit tests for TypeScript/React codebases using Vitest, @testing-library/react, and @testing-library/user-event. Orchestrates investigation, planning, implementation (via Coder subagent), and review (via Code Review subagent) in an iterative loop. Also provides testing guidance for any agent writing or reviewing tests. Triggers on: 'write tests for', 'add tests', 'test coverage', 'unit test', '/unit-test', or when an agent needs testing decision guidance."
---

# Unit Test Skill

Orchestrate unit test creation via Coder and Code Review subagents, or provide testing guidance to any agent.

## When referenced by a subagent

Read `references/testing-guidance.md` for testing decisions: scenario identification, setUpTest pattern selection, query priority, file structure, checklists, and anti-patterns.

## When orchestrating test creation

You are a **coordination-only** orchestrator. You MUST NOT write or modify any code directly. Plan unit tests, get user approval, then orchestrate implementation across Coder and Code Review subagents while keeping your own context window minimal.

### Input

Parse from the user's request:

- **target**: File(s) or component(s) to test (required)
- **coverage**: `full` (default) or `quick`. Full = target 100% coverage. Quick = cover critical paths only.
- **use case**: new tests, refactoring existing tests, improving coverage, or filling coverage gaps — infer from context

### Phase 1: Investigate

Use an Explore subagent to:

1. Read the target source file(s)
2. Find existing tests for the target
3. Read 2-3 nearby test files for local patterns (setUpTest variation, selectors, mocks)
4. Check `src/test-utils/` for available helpers
5. Identify the scenario type: component / store / utility / API
6. If existing tests found, run: `yarn test:coverage [test-file] --coverage.include='[source-file]'`
7. Determine use case: new tests, refactoring, improvement, or coverage gaps

Return a concise summary of findings. Do NOT load full file contents into your own context — only the pattern observations, scenario type, and key file paths.

### Phase 2: Plan

Based on the investigation, produce a test plan **directly in your response** as formatted text. Never write the plan to a file.

Include:

```
## Test Plan: [target]

**Suite**: `[suite name]` ([create new | add to existing])
**Coverage mode**: [full | quick]
**Type**: [component | store | utility | API]
**setUpTest variation**: [deferred render | store-only | hierarchical | pre-loaded]

### Tests to write/change:
1. must [behavior] when [condition] — tests [what]
2. must [behavior] when [condition] — tests [what]
...

### Mocks needed:
- [mock description]

### Edge cases:
- [edge case]
```

For **refactoring**: describe what changes and why.
For **coverage gaps**: map specific uncovered lines/branches to tests that will cover them.

#### Resolve ambiguities first

If the investigation surfaced any ambiguities, unknowns, or decisions that require user input, use `AskUserQuestion` to resolve **every one of them** before presenting the plan. Do not list unresolved questions in the plan — resolve them now.

#### Present the plan and get approval

Output the plan then use `AskUserQuestion` to ask the user to approve, request changes, or reject.

**Do NOT proceed to Phase 3 until the user has explicitly approved the plan.**

### Phase 3: Execute

Coder -> Reviewer loop, max 3 iterations.

#### Dispatch Coder

Launch a Coder subagent with a focused brief containing ONLY:

- The approved test plan
- Target source file path(s) and test file path
- Existing patterns discovered in Phase 1 (nearby test examples, test-utils available)
- Instruction: **Read `references/testing-guidance.md` (relative to this skill) for testing decision guidance. Follow it.**

#### Dispatch Reviewer

After Coder completes, launch a Code Review subagent with:

- The test file that was written/modified
- The source file(s) being tested
- Instruction: **Read `references/testing-guidance.md` (relative to this skill) for testing decision guidance. Verify adherence.**
- Review checklist:
  - Correctness: do tests verify the described behavior?
  - Testing Library best practices (role queries, userEvent, no implementation testing)
  - Project conventions (setUpTest pattern, element selectors, naming)
  - Missing edge cases or branches (especially in `full` coverage mode)
  - Anti-patterns: snapshot tests, manual DOM traversal, excessive mocking, testing framework internals

#### Iterate

If the reviewer flags **major issues** (incorrect assertions, missing critical tests, anti-patterns, convention violations):

1. Dispatch Coder again with the specific review feedback
2. Dispatch Reviewer again to verify fixes
3. Max 3 iterations — if the same issue persists after 3, surface it to the user

**Stop iterating** when the reviewer reports only minor/cosmetic issues or no issues.

### Phase 4: Verify + Report

#### Verify

1. Run tests: `yarn test [test-file]`
2. If `full` coverage mode, run:
   ```
   yarn test:coverage [test-file] --coverage.include='[source-file]'
   ```
   Multiple source files use repeated flags:
   ```
   yarn test:coverage [test-file] --coverage.include='[file1]' --coverage.include='[file2]'
   ```
3. Fix failures — dispatch Coder with failure output, review, re-run (max 3 attempts)
4. If `full` coverage with gaps: dispatch Coder for additional tests to cover uncovered lines, review, re-verify

#### Report

Present structured results:

```
## Test Results: [suite name]

**Status**: [pass | fail]
**Coverage mode**: [full | quick]

### Tests written ([count]):
- [test name]
- [test name]
...

### Coverage ([full only]):
| Metric     | % |
|------------|---|
| Statements | X |
| Branches   | X |
| Functions  | X |
| Lines      | X |

### Uncovered lines ([if any]):
- [file:line — reason]
```

## Rules

- **Never write code yourself** — all implementation and review happens via subagents
- **Minimise your context** — pass subagents only what they need; summarise, don't copy
- **Log progress** — after each phase transition, briefly note what completed and what is dispatching next
- **Prefer small briefs** — focused subagent tasks over large monolithic ones
- **User checkpoints** — present the plan before executing; surface persistent issues rather than looping forever
- **Handle failures** — if a subagent fails or gets stuck after 3 attempts, report to the user and ask how to proceed
- **Plan in chat, not in files** — always output the plan directly in your response text; never write it to a file
- **Resolve before executing** — use AskUserQuestion to resolve every ambiguity before starting Phase 3; never proceed with unresolved questions
- **Explicit approval required** — do not start execution until the user has explicitly approved the plan via AskUserQuestion
