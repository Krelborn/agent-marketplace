# Testing Principles

Based on Kent C. Dodds' testing philosophy adapted for Vitest + Testing Library.

## Core Principle

> The more your tests resemble the way your software is used, the more confidence they can give you.

Test user behavior, not implementation details. If a user can't observe it, don't test it.

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

## Test Structure

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

## Arrange-Act-Assert Pattern

- Organize individual tests with AAA pattern using vertical whitespace to separate sections
- NO section comments needed - the whitespace structure is self-evident
- Pattern:
  - Setup (arrange)
  - Execute action (act)
  - Verify result (assert)
- Improves test readability and maintainability

## setUpTest Pattern

A test setup helper that prepares mocks, stores, and a render function. **It does NOT call render itself.** Tests control rendering explicitly.

```typescript
function setUpTest(options = {}) {
  // 1. Create mocks, stores, test data
  // 2. Configure mock behavior
  // 3. Optionally create a render() function
  return { store, render, ...handlers };
}
```

### Store Tests

No rendering — return store + dependencies.

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

### Component Tests — Deferred Render (most common)

Returns an async `render()` function. Tests call it when ready.

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

### Hierarchical Helpers

For complex domains, layer complexity progressively. Each helper builds on the previous:

```typescript
function setUpTest() { ... }
function setUpTestWithItem() { return { ...setUpTest(), item, store }; }
function setUpTestWithEditor() { return { ...setUpTestWithItem(), editor }; }
```

Tests pick the level of setup they need.

### Key Conventions

- **Naming**: `setUpTest`, `setUpTestWith[Feature]`, `setUpLoadedTest`
- **Parameters**: single options object with defaults: `{ item = null, isReadOnly = false } = {}`
- **Return**: always a named object — `return { store, render }`
- **Render control**: tests call `render()` — setup never renders
- **Async pairing**: for pre-loaded state, pair sync `setUpTest` with async `setUpLoadedTest` that awaits loading before returning
- **Async waits**: return `waitFor*` helpers using MobX `when()` or Testing Library `waitFor()`
- **Mock reset**: `vi.resetAllMocks()` at start when needed

## Naming Convention

Use `must [behavior] when [condition]` pattern:

```typescript
test("must show error message when email is invalid", ...);
test("must disable submit button while request is pending", ...);
test("must call onSave with updated values when form is submitted", ...);
test("must render empty state when no items exist", ...);
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

## Anti-Patterns to Avoid

1. **Snapshot tests** — brittle, low signal, hard to review
2. **Testing implementation** — `wrapper.instance()`, internal state checks
3. **Manual DOM traversal** — `container.querySelector(".my-class")`
4. **Excessive mocking** — mock boundaries (APIs, stores), not internals
5. **Copy-paste tests** — extract `setUpTest()` helpers instead
6. **Testing library code** — don't test MobX reactivity or React rendering itself
7. **Asserting on mock call counts** — assert on visible outcomes instead when possible
