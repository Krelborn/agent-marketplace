---
name: unit-test
description: "Write unit tests for React components and TypeScript code using Vitest, @testing-library/react, and @testing-library/user-event. Use when the user asks to: write tests, add test coverage, create a test suite, test a component, test a store, test a utility, or achieve coverage targets. Supports specifying test suite names, 100% coverage by default with opt-out, and follows Kent C. Dodds testing principles. Triggers on phrases like 'write tests for', 'add tests', 'test coverage', 'unit test', '/unit-test'."
---

# Unit Test

Write unit tests following Kent C. Dodds testing principles with Vitest and Testing Library.

## Arguments

Parse these from the user's request:

- **target**: File(s) or component(s) to test (required)
- **suite**: Test suite name to create or add tests to (optional - inferred from target if omitted)
- **coverage**: `full` (default) or `quick`. Full = target 100% coverage. Quick = cover critical paths only.

## Workflow

### Phase 1: Discover

1. Read the target source file(s) thoroughly
2. Read the project's CLAUDE.md for testing conventions
3. Find and read 2-3 existing test files near the target to learn local patterns:
   - `setupTest()` / `setUpTest()` helper pattern
   - Element selector objects (`const elements = { get foo() { ... } }`)
   - Mock store setup
   - Import conventions
4. Read test utilities: check `src/test-utils/` for available helpers
5. Identify the testing scenario:
   - **React component** → use `renderWithRootStore()`, `userEvent`, role queries
   - **MobX store** → use `MockRootStore`, `when()` from MobX for async
   - **Utility function** → direct unit tests, no rendering needed
   - **API layer** → use `MockSignals` pattern

### Phase 2: Plan

Present a test plan to the user before writing. Include:

```
## Test Plan: [target]

**Suite**: `[suite name]` ([create new | add to existing])
**Coverage mode**: [full | quick]
**Coverage command**: `yarn test:coverage [test-file] --coverage.include='[source-file]'`
**Type**: [component | store | utility | API]

### Tests to write:
1. [test description] - tests [what behavior]
2. [test description] - tests [what behavior]
...

### Mocks needed:
- [mock description]

### Edge cases:
- [edge case]
```

Wait for user approval before proceeding.

### Phase 3: Write → Review → Iterate

Use the **coder** and **code-reviewer** sub-agents in a loop:

#### Step 1: Write (coder agent)

Launch the `coder` sub-agent via the Task tool with `subagent_type: "coder"`. Include in the prompt:

- The approved test plan from Phase 2
- The target source file path(s)
- The test file path to create/edit
- The testing rules below
- The contents of `../../docs/testing-principles.md`
- The existing test patterns discovered in Phase 1 (nearby test examples, test-utils available)

**Testing rules for the coder prompt:**

- Query by role, label, placeholder, text — never by class/id/test-id
- Test user-visible behavior, not implementation
- Use `userEvent` (not `fireEvent`) for interactions
- One assertion concept per test (multiple `expect` OK if testing one behavior)
- Use `setupTest()` pattern for shared setup
- Use element selector objects for repeated queries
- Flat test structure — avoid nested `describe` blocks. One `describe` per suite max.
- Follow existing project conventions discovered in Phase 1

**File structure template to include in coder prompt:**

```typescript
// Imports

// Mocks (vi.mock calls — hoisted, declare before imports)

// Types/interfaces if needed

// Setup helper
function setupTest(overrides?: Partial<Props>) {
  const defaults = {
    /* sensible defaults */
  };
  const props = { ...defaults, ...overrides };
  // render or instantiate
  return {
    /* useful references */
  };
}

// Element selectors (for component tests)
const elements = {
  get submitButton() {
    return screen.getByRole("button", { name: "Submit" });
  },
};

// Tests
describe("ComponentName", () => {
  test("must [expected behavior] when [condition]", async () => {
    // Arrange
    // Act
    // Assert
  });
});
```

#### Step 2: Review (code-reviewer agent)

After the coder finishes, launch the `code-reviewer` sub-agent via the Task tool with `subagent_type: "code-reviewer"`. Include in the prompt:

- The test file that was written/modified
- The source file(s) being tested
- Ask it to review for:
  - Correctness: do tests actually verify the described behavior?
  - Testing Library best practices (role queries, userEvent, no implementation testing)
  - Project convention adherence (setupTest pattern, element selectors, naming)
  - Missing edge cases or branches (especially if `full` coverage mode)
  - Anti-patterns: snapshot tests, manual DOM traversal, excessive mocking, testing framework internals

#### Step 3: Iterate

If the reviewer reports **major issues** (incorrect assertions, missing critical tests, anti-patterns, convention violations):

1. Launch the `coder` agent again with the review feedback to fix the issues
2. Launch the `code-reviewer` agent again to verify fixes
3. Repeat until no major issues remain

**Stop iterating** when the reviewer reports only minor/cosmetic issues or no issues. Max 3 iterations to avoid unbounded loops.

### Phase 4: Verify

1. Run the tests: `yarn test [test-file-pattern]`
2. If coverage mode is `full`, run coverage scoped to the source files under test:
   ```
   yarn test:coverage [test-file] --coverage.include='[source-file]'
   ```
   Multiple source files use repeated flags:
   ```
   yarn test:coverage [test-file] --coverage.include='[file1]' --coverage.include='[file2]'
   ```
   Example: testing `DrawingEditor.tsx` via its test suite:
   ```
   yarn test:coverage src/DrawingEditor/DrawingEditor.test.ts --coverage.include='src/DrawingEditor/DrawingEditor.tsx'
   ```
   This ensures coverage reports only the files the suite is responsible for, not the entire project.
3. Fix any failures — re-run until green
4. If `full` coverage: check uncovered lines, add tests, repeat until 100%

### Phase 5: Report

Present results:

```
## Test Results: [suite name]

**Status**: [pass | fail]
**Coverage mode**: [full | quick]

### Tests written ([count]):
- ✓ [test name]
- ✓ [test name]
...

### Coverage ([full only]):
| Metric     | % |
|------------|---|
| Statements | X |
| Branches   | X |
| Functions  | X |
| Lines      | X |

### Uncovered lines ([if any]):
- [file:line - reason]
```

## References

For Kent C. Dodds testing principles and Testing Library best practices, read `../../docs/testing-principles.md`.
