---
name: unit-testing-guide
description: >-
  Loads project testing standards for writing and reviewing Vitest unit tests
  in TypeScript/React codebases. Covers Testing Library query priority, setUpTest
  pattern selection, test naming conventions, async patterns, and anti-patterns.
  Use this skill whenever you are: writing new test files or adding tests to
  existing files, reviewing or giving feedback on test code, modifying .test.ts
  or .test.tsx files, choosing between setUpTest variations, deciding how to
  query or assert on rendered components, or fixing test failures related to
  structure or conventions. This applies even for small test changes — loading
  these standards ensures consistency. If you are touching test code in any
  capacity, check this skill.
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
