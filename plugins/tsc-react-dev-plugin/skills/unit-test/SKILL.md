---
name: unit-test
description: "Guidance for writing unit tests in TypeScript/React codebases using Vitest, @testing-library/react, and @testing-library/user-event. Reference this skill when writing tests, adding test coverage, creating test suites, or reviewing test code. Covers scenario identification, setUpTest pattern selection, file structure, and practical checklists. Triggers on phrases like 'write tests for', 'add tests', 'test coverage', 'unit test', '/unit-test'."
---

# Unit Test Knowledge

Practical guidance for writing unit tests. For the theoretical foundation and full code examples, see `../../docs/testing-principles.md`.

## Scenario Identification

| Scenario | Signals | setUpTest variation | Key tools |
|---|---|---|---|
| **React component** | `.tsx` file, JSX, renders UI | Deferred render | `renderWithRootStore()`, `userEvent`, role queries |
| **MobX store** | `makeObservable`, `@observable`, `@action` | Store-only | `MockRootStore`, `when()` from MobX |
| **Utility function** | Pure function, no React/MobX imports | Store-only (or none) | Direct calls, no rendering |
| **API layer** | API calls, signal handling, fetch/axios | Store-only | `MockSignals` pattern, mock API responses |

## setUpTest Decision Guide

Choose the variation that matches your test's needs:

**Store-only** — Use when there's no UI to render. Return store + dependencies directly.
- Store tests, utility tests, API layer tests
- No `render()` function returned

**Deferred render** — Most common for components. Return an async `render()` that tests call explicitly.
- Component tests where tests need to configure state before rendering
- Tests call `await render()` when ready

**Hierarchical** (`setUpTestWith[Feature]`) — Use when a domain has multiple layers of complexity.
- Complex domains with shared base setup and feature-specific extensions
- Each helper builds on the previous: `setUpTest` → `setUpTestWithItem` → `setUpTestWithEditor`
- Tests pick the level they need

**Pre-loaded state** — Pair sync `setUpTest` + async `setUpLoadedTest` when tests need data already loaded.
- Scenarios where some tests need pre-loading and others test the loading itself
- `setUpLoadedTest` awaits async operations before returning

See `../../docs/testing-principles.md` § setUpTest Pattern for full code examples of each variation.

## File Structure Template

```typescript
// Imports

// Mocks (vi.mock calls — hoisted, declare before imports)

// Types/interfaces if needed

// Tests
describe("ComponentName", () => {
  test("must [expected behavior] when [condition]", async () => {
    // Arrange
    // Act
    // Assert
  });
});

// Setup helper — deferred render variation (most common for components)
// See ../../docs/testing-principles.md for store-only, immediate-render, and hierarchical variations
function setUpTest({ /* option = default */ } = {}) {
  // Create mocks, stores, test data
  const store = new MockStore();

  const render = async () => {
    renderTL(/* <Provider><Component /></Provider> */);
    await screen.findByRole("region", { name: "..." });
  };

  return { store, render };
}

// Element selectors (for component tests)
const elements = {
  get submitButton() {
    return screen.getByRole("button", { name: "Submit" });
  },
};
```

## Testing Rules Checklist

- [ ] Query by role, label, placeholder, text — never by class/id/test-id (see query priority in `../../docs/testing-principles.md`)
- [ ] Test user-visible behavior, not implementation details
- [ ] Use `userEvent` (not `fireEvent`) for all interactions
- [ ] One assertion concept per test (multiple `expect` OK if testing one behavior)
- [ ] Flat test structure — one `describe` per suite max, no nesting
- [ ] Name tests `must [behavior] when [condition]`
- [ ] Use `setUpTest()` pattern for shared setup (see decision guide above)
- [ ] Use element selector objects for repeated queries
- [ ] Follow existing project conventions from nearby test files

## Coverage Commands

Run tests:
```
yarn test [test-file-pattern]
```

Run scoped coverage:
```
yarn test:coverage [test-file] --coverage.include='[source-file]'
```

Multiple source files use repeated flags:
```
yarn test:coverage [test-file] --coverage.include='[file1]' --coverage.include='[file2]'
```

Example:
```
yarn test:coverage src/DrawingEditor/DrawingEditor.test.ts --coverage.include='src/DrawingEditor/DrawingEditor.tsx'
```

## Anti-Pattern Reminders

- **No snapshot tests** — brittle, low signal, hard to review
- **No implementation testing** — don't check internal state, `wrapper.instance()`, or private fields
- **No manual DOM traversal** — never use `container.querySelector(".my-class")`
- **No excessive mocking** — mock at boundaries (APIs, stores), not internals
- **No copy-paste tests** — extract `setUpTest()` helpers instead
- **No testing library code** — don't test MobX reactivity or React rendering itself
- **No asserting on mock call counts** — prefer asserting on visible outcomes
