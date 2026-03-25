# Testing Guidance

Practical guidance for writing unit tests in TypeScript/React codebases using Vitest, @testing-library/react, and @testing-library/user-event.

## Core Principle

> The more your tests resemble the way your software is used, the more confidence they can give you.

Test user behavior, not implementation details. If a user can't observe it, don't test it.

## Scenario Identification

| Scenario             | Signals                                    | setUpTest variation  | Key tools                                          |
| -------------------- | ------------------------------------------ | -------------------- | -------------------------------------------------- |
| **React component**  | `.tsx` file, JSX, renders UI               | Deferred render      | `renderWithRootStore()`, `userEvent`, role queries |
| **MobX store**       | `makeObservable`, `@observable`, `@action` | Store-only           | `MockRootStore`, `when()` from MobX                |
| **Utility function** | Pure function, no React/MobX imports       | Store-only (or none) | Direct calls, no rendering                         |
| **API layer**        | API calls, signal handling, fetch/axios    | Store-only           | `MockSignals` pattern, mock API responses          |

## Query Priority

Use queries in this order (most preferred first):

1. `getByRole` — accessible roles (button, textbox, heading, dialog, etc.)
2. `getByLabelText` — form fields
3. `getByPlaceholderText` — when label unavailable
4. `getByText` — non-interactive elements
5. `getByDisplayValue` — filled form elements
6. `getByAltText` — images
7. `getByTitle` — rarely needed
8. **Never**: `getByTestId`, class selectors, DOM structure queries

Use `name` option with `getByRole` for specificity:

```typescript
screen.getByRole("button", { name: "Save" });
screen.getByRole("textbox", { name: "Email" });
screen.getByRole("dialog", { name: "Confirm Delete" });
```

## Interaction

Always use `@testing-library/user-event` over `fireEvent`:

```typescript
import userEvent from "@testing-library/user-event";

// Good
await userEvent.click(button);
await userEvent.type(input, "text");
await userEvent.clear(input);
await userEvent.selectOptions(select, "option");
await userEvent.keyboard("{Enter}");

// Bad — fireEvent doesn't simulate real user behavior
fireEvent.click(button);
fireEvent.change(input, { target: { value: "text" } });
```

## Async Patterns

```typescript
// Wait for element to appear
await waitFor(() => {
  expect(screen.getByText("Loaded")).toBeInTheDocument();
});

// Wait for element to disappear
await waitFor(() => {
  expect(screen.queryByText("Loading...")).not.toBeInTheDocument();
});

// MobX observable changes
import { when } from "mobx";
await when(() => store.isLoaded);

// Controlled async with manual promises
const { promise, resolver } = createManualPromise<string>();
mockFn.mockReturnValueOnce(promise);
// ... trigger action ...
await act(async () => {
  resolver("result");
  await promise;
});
```

## Test Structure & Naming

Flat structure, no nesting beyond one `describe`:

```typescript
// Good — flat
describe("LoginForm", () => {
  test("must show error when email is empty", async () => { ... });
  test("must submit when form is valid", async () => { ... });
  test("must disable button while submitting", async () => { ... });
});

// Bad — nested describes
describe("LoginForm", () => {
  describe("validation", () => {
    describe("email", () => {
      test("shows error", () => { ... });
    });
  });
});
```

Use `must [behavior] when [condition]` pattern:

```typescript
test("must show error message when email is invalid", ...);
test("must disable submit button while request is pending", ...);
test("must call onSave with updated values when form is submitted", ...);
test("must render empty state when no items exist", ...);
```

## setUpTest Pattern

A test setup helper that prepares mocks, stores, and a render function. **It does NOT call render itself.** Tests control rendering explicitly.

### setUpTest Decision Guide

**Store-only** — No UI to render. Return store + dependencies directly.

```typescript
function setUpTest() {
  const store = new MockStore();
  MockAPI.getItems.mockResolvedValue(sampleItems);
  return { store };
}

test("must create item", async () => {
  const { store } = setUpTest();
  await store.create({ name: "test" });
  expect(MockAPI.create).toHaveBeenCalledTimes(1);
});
```

**Deferred render** (most common for components) — Returns async `render()`. Tests call it when ready.

```typescript
function setUpTest({ item = null } = {}) {
  const store = new MockStore();
  store.item = item;

  const render = async () => {
    renderTL(
      <StoreContext.Provider value={store}>
        <MyComponent store={store} />
      </StoreContext.Provider>
    );
    await screen.findByRole("region", { name: "Editor" });
  };

  return { store, render };
}

test("must render editor", async () => {
  const { render } = setUpTest();
  await render();
  expect(screen.getByRole("region", { name: "Editor" })).toBeInTheDocument();
});
```

Note: `renderTL` is Testing Library's `render`, renamed to avoid collision with the local `render` function.

**Hierarchical** (`setUpTestWith[Feature]`) — Complex domains with shared base setup and feature-specific extensions.

```typescript
function setUpTest() { ... }
function setUpTestWithItem() { return { ...setUpTest(), item, store }; }
function setUpTestWithEditor() { return { ...setUpTestWithItem(), editor }; }
```

Tests pick the level of setup they need.

**Pre-loaded state** — Pair sync `setUpTest` + async `setUpLoadedTest` when tests need data already loaded.

### Key Conventions

- **Naming**: `setUpTest`, `setUpTestWith[Feature]`, `setUpLoadedTest`
- **Parameters**: single options object with defaults: `{ item = null, isReadOnly = false } = {}`
- **Return**: always a named object — `return { store, render }`
- **Render control**: tests call `render()` — setup never renders
- **Async pairing**: for pre-loaded state, pair sync `setUpTest` with async `setUpLoadedTest` that awaits loading before returning
- **Async waits**: return `waitFor*` helpers using MobX `when()` or Testing Library `waitFor()`
- **Mock reset**: `vi.resetAllMocks()` at start when needed

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
function setUpTest({
  /* option = default */
} = {}) {
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

## What to Test

**Do test:**

- User-visible output (text, elements appearing/disappearing)
- User interactions (click, type, select)
- Accessibility (elements findable by role)
- Error states users can see
- Loading/disabled states
- Callback invocations with correct args

**Don't test:**

- Internal state values directly
- Implementation methods
- CSS classes or styling
- Component lifecycle
- Private store fields (test via public behavior)
- Things the framework guarantees

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

## Anti-Patterns

1. **No snapshot tests** — brittle, low signal, hard to review
2. **No implementation testing** — don't check internal state, `wrapper.instance()`, or private fields
3. **No manual DOM traversal** — never use `container.querySelector(".my-class")`
4. **No excessive mocking** — mock at boundaries (APIs, stores), not internals
5. **No copy-paste tests** — extract `setUpTest()` helpers instead
6. **No testing library code** — don't test MobX reactivity or React rendering itself
7. **No asserting on mock call counts** — prefer asserting on visible outcomes

## Checklist

- [ ] Query by role, label, placeholder, text — never by class/id/test-id
- [ ] Test user-visible behavior, not implementation details
- [ ] Use `userEvent` (not `fireEvent`) for all interactions
- [ ] One assertion concept per test (multiple `expect` OK if testing one behavior)
- [ ] Flat test structure — one `describe` per suite max, no nesting
- [ ] Name tests `must [behavior] when [condition]`
- [ ] Use `setUpTest()` pattern for shared setup
- [ ] Use element selector objects for repeated queries
- [ ] Follow existing project conventions from nearby test files
