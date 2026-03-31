---
name: component-review
description: "Perform a detailed review of React components for quality, simplicity, composability, and best practices. Use when: (1) the user asks to review React components, (2) after implementing or refactoring React components via the implement skill, (3) the user asks '/component-review', or (4) when dispatched as a specialized reviewer during implementation. Covers DRY composition, props documentation, accessibility, component design, and general React quality — skipping anything already enforced by linters."
---

# Component Review

Perform a focused review of React components for quality, simplicity, and composability.

## Scope

Determine what to review:

1. **Default — changed code**: If not on `main`, diff all commits on the current branch against `main` (`git diff main...HEAD`). If on `main`, diff against `HEAD~1`. Filter to `.tsx` files.
2. **User-specified scope**: If the user names specific files, directories, or components, review those instead.
3. **Extended scope**: When reviewing changed components, also read their direct imports and consumers (one level) to identify cross-component duplication opportunities. Do not review these surrounding files in full — only scan for duplication with the changed components.

Use `git diff` to identify changed files, then `Read` each component file. For extended scope, use `Grep` to find imports/usages.

## Review Checklist

Evaluate each component against the principles in [references/react-component-quality.md](references/react-component-quality.md). Read that file before starting the review.

## Output Format

### Summary

1-3 sentence overall assessment.

### Issues Found

For each issue:

- **Severity**: Critical | Major | Minor | Suggestion
- **Principle**: Which principle from the checklist
- **Location**: File and line/area
- **Problem**: What's wrong
- **Fix**: Specific recommendation (show code if helpful)

Group by file when reviewing multiple components.

### What's Done Well

2-3 things the components do well. Balanced review matters.

### Verdict

One of: **Approve** | **Approve with suggestions** | **Request changes**

## Guidelines

- Skip anything enforced by linters (exhaustive deps, import order, unused vars, etc.)
- Read existing codebase patterns before flagging something as wrong
- Lead with the most impactful issues
- Show concrete code examples for DRY/composition suggestions — don't just say "extract a component", show what it would look like
- If components are clean and well-designed, say so. Don't manufacture issues
- When the review is dispatched by the implement skill, focus findings on actionable changes — omit suggestions that would require rearchitecting beyond the current task's scope
