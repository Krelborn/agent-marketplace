---
name: review-and-fix
description: "User-driven code review and fix workflow. The user identifies specific code issues — bugs, type problems, design concerns, duplication, missing validation, etc. — and the agent evaluates each one, asks clarifying questions, presents fix options with recommendations, then implements all agreed fixes via subagents. Use this skill whenever the user says they have issues to walk through, problems to discuss, concerns about specific code, or feedback they want to go over before fixing. Covers single issues ('this line is wrong'), multiple issues ('I have 3 problems to go through'), and design-level concerns ('the whole approach feels wrong'). Key signal: the user wants to discuss the fix approach before implementation, not just have code written or reviewed automatically. Do NOT use for: general PR reviews (use code-reviewer), direct fix requests where the user already knows the solution, new feature implementation (use coder/implement), or automated component quality checks (use component-review)."
---

# Review and Fix

An interactive code review session where the user identifies issues, the agent evaluates and proposes solutions, and then all agreed fixes are implemented via subagents.

The workflow has three phases: **Collect**, **Plan**, **Execute**.

## Phase 1: Collect Issues

This phase is a conversation loop. Work through one issue at a time with the user until they confirm there are no more.

### Starting the session

Ask the user to describe the first issue. Prompt them naturally — they might point to a specific line, a range, a file, or a broader design concern. Accept any of these.

### For each issue

1. **Read the code** the user is referring to. Use `Read` to load the file(s) and understand the surrounding context — not just the exact lines mentioned, but enough to see how the code fits into its module.

2. **Evaluate against the user's feedback.** Assess whether the problem the user describes is valid. If you agree, say so and explain why. If you see nuance the user may have missed (e.g., the code looks wrong but is actually correct for a subtle reason), raise it — but don't be defensive of bad code. Be honest.

3. **Ask clarifying questions** if the issue is ambiguous. Examples of useful clarifications:
   - "Are you concerned about the type safety here, or the runtime behavior?"
   - "Should this fix preserve backwards compatibility with callers, or can we change the interface?"
   - "Is this a bug you've hit in practice, or a code quality concern?"

   Don't ask questions you can answer by reading the code. Only ask when the user's intent or constraints are genuinely unclear.

4. **Present fix options.** Offer 2-3 concrete approaches where there's a meaningful choice to make. For each option:
   - Describe the approach in plain terms
   - Show a brief code sketch if it helps (keep it short — this isn't the implementation phase)
   - State trade-offs: complexity, scope of change, risk, performance implications
   - Give your recommendation and explain why

   If there's only one sensible fix, say so — don't manufacture fake alternatives. Just present the fix and confirm the user is happy with it.

5. **Record the agreed fix.** Once the user picks an approach (or agrees with your recommendation), note it internally. Track:
   - File(s) and location
   - Problem summary
   - Agreed solution approach

6. **Ask if there are more issues.** A simple "Are there other issues to look at, or shall we move on to fixing these?" Keep it natural.

### When collection is complete

Summarize all collected issues in a numbered list before proceeding:

```
## Issues to address

1. **[File:lines]** — [Problem summary] → [Agreed fix approach]
2. **[File:lines]** — [Problem summary] → [Agreed fix approach]
...
```

Ask the user to confirm the list is correct and complete. If they add or change anything, update accordingly.

## Phase 2: Plan

Use plan mode (`EnterPlanMode`) to create a structured implementation plan. The plan should:

- Group fixes that touch the same file or module together as a single unit of work
- Identify dependencies between fixes (e.g., a type change that affects multiple call sites)
- Mark independent units that can run in parallel
- Order dependent work correctly — foundational changes (types, interfaces, utilities) before consumers
- Include verification steps (type checker, tests, linter) as the final step

Present the plan to the user for approval before proceeding. Exit plan mode (`ExitPlanMode`) once approved.

## Phase 3: Execute

You are a **coordination-only** orchestrator in this phase. Do not write or modify code directly — all implementation happens via subagents.

### For each unit of work

**1. Dispatch Coder** — Launch a coder subagent (`subagent_type: "tsc-react-dev-plugin:coder"`) with a brief containing:

- "The plan is already complete. Do NOT plan or outline an approach — skip straight to reading the relevant files and writing code. Every response must include actual code changes via Edit/Write tools."
- The specific fix(es) for this unit, including the problem description and agreed solution
- Relevant file paths and line numbers
- Acceptance criteria derived from the agreed fix
- Context from any prior completed units if relevant

**2. Dispatch Reviewer** — When the coder completes, launch a code review subagent (`subagent_type: "tsc-react-dev-plugin:code-reviewer"`) to verify:

- The fix matches the agreed solution from Phase 1
- No regressions or new issues introduced
- Adherence to project coding standards
- Code quality and correctness

**3. Iterate if needed** — If the reviewer flags issues, pass specific feedback back to a fresh coder subagent. Repeat until approved. Cap at 3 iterations per unit — if the same issue persists, surface it to the user with the reviewer's feedback and ask how to proceed.

**4. Parallelism** — Run independent units concurrently. Only serialize units with dependencies.

### Verification

After all units complete, run the verification steps from the plan (type checker, test suite, linter). If verification fails, dispatch a coder subagent with the failure output, review the fix, re-verify. Max 3 attempts, then report to the user.

### Final report

Present a structured summary:

```
## Review and Fix Complete

**Status**: [pass | fail | partial]
**Issues resolved**: [N/M]

### Changes made
- **Issue 1** — [File(s)]: [Brief description of what changed]
- **Issue 2** — [File(s)]: [Brief description of what changed]
...

### Verification results
- Type checker: [pass | fail]
- Tests: [pass | fail] ([count] tests)
- Linter: [pass | fail]

### Unresolved issues (if any)
- [Description and why it couldn't be resolved]
```

## Guidelines

- **Stay in the user's frame.** During Phase 1, the user is the expert on what's wrong. Your job is to understand their concern, validate it, and help shape the fix — not to redirect them to different issues you spotted.
- **Don't scope-creep.** If you notice other problems in the code during Phase 1, don't bring them up unless they're directly related to the issue being discussed. The user is driving.
- **Be direct about trade-offs.** When presenting options, don't hedge — give a clear recommendation. The user can override you.
- **Keep Phase 1 conversational.** This isn't a form to fill out. Adapt to how the user describes issues — some will give you exact line numbers, others will wave vaguely at a file. Meet them where they are.
- **Escalate, don't loop.** If a coder/reviewer cycle isn't converging after 3 iterations, stop and ask the user rather than trying a 4th time.
