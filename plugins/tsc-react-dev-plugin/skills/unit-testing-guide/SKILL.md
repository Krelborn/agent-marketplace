---
name: unit-testing-guide
description: >-
  Use this skill whenever work involves or should involve Vitest unit tests
  for TypeScript/React code. This includes writing new tests, adding test cases,
  refactoring or restructuring existing test files, reviewing test code quality,
  migrating test patterns (e.g. fireEvent to userEvent, nested describes to flat),
  fixing act() warnings or async test issues, replacing snapshots with behavioral
  assertions, updating queries to use proper roles, and any modification to
  .test.ts or .test.tsx files no matter how small. Also use this skill when
  writing or refactoring React components (their tests likely need updating),
  creating MobX stores (they need tests), reviewing PRs that include test files,
  or adding accessibility roles to components (which changes how tests query them).
  If there is even a possibility that test code will be read, written, modified,
  or reviewed as part of the current task, use this skill. Provides project
  testing standards covering Testing Library query priority, setUpTest patterns,
  test naming conventions, async patterns, and anti-patterns.
---

# Unit Testing Guide

Load and apply project testing standards when writing or reviewing Vitest unit tests.

## What to do

Read [testing-guidance.md](references/testing-guidance.md). It contains the full testing standards covering:

- **Scenario identification** — classifying what you're testing (component, store, utility, API) and which setUpTest variation fits
- **Query priority** — which Testing Library queries to prefer (getByRole first, never getByTestId)
- **Screen usage** — always use `screen` for queries, never destructure from `render()`
- **setUpTest patterns** — deferred render, store-only, hierarchical, and pre-loaded variations with decision guidance
- **Test naming** — the `must [behavior] when [condition]` convention
- **Interaction** — always `userEvent.setup()`, never fireEvent
- **Async patterns** — prefer `findBy*`, use `waitFor` only for complex assertions, avoid unnecessary `act()`
- **File structure** — imports, mocks, describe block, setup helpers, element selectors
- **What to test / what not to test** — user-visible behavior yes, implementation details no
- **Anti-patterns** — snapshots, manual DOM traversal, excessive mocking, regex for known strings, manual cleanup
- **Checklist** — final verification before considering tests complete

Apply these standards to your current task. They are not suggestions — they are project conventions that ensure test quality and consistency across the codebase.
