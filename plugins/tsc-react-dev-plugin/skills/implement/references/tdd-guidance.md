# TDD Workflow

Test-driven development is the default approach for testable units of work. Follow Red → Green → Refactor.

## When to apply TDD

Apply TDD when the work is:

- Business logic or pure functions
- MobX stores, custom hooks, API clients, utilities
- A bug fix — reproduce the bug as a failing test before fixing it
- Anything with clearly defined inputs and outputs

Skip TDD when the work is:

- Pure styling, layout, or visual-only changes
- Exploratory spikes intended to be discarded
- Trivial config edits or pure type-only refactors that no test could meaningfully exercise
- Modifications to code already covered by existing tests that exercise the changed behavior (extend the existing tests instead)

When unsure, default to TDD. The cost of writing the test first is small, and the test stays as a regression net.

## The cycle

### 1. RED — write a failing test

- Write the smallest test that exercises the new behavior, or reproduces the bug.
- Run the test. Confirm it fails.
- Confirm it fails **for the right reason** — the expected assertion is unmet, not a syntax error, missing import, or mistyped path. A test that fails for the wrong reason proves nothing.
- Add one failing test at a time. Do not stack multiple failing tests before writing implementation.

### 2. GREEN — make the test pass with minimum code

- Write the simplest implementation that makes the test pass. No premature abstraction. No speculative features.
- Never modify the test to fit a broken implementation — the test is the contract.
- Run all tests in the affected file(s). Every test must pass before moving on.

### 3. REFACTOR — clean up while green

- With tests green, improve the code: extract helpers, rename for clarity, remove duplication.
- Re-run tests after every refactor step. They must stay green.
- Do not add new behavior during refactor. New behavior starts the next RED step.

## Working through a unit

For new behavior, loop:

1. Pick the next thinnest behavior to add.
2. RED — write the failing test.
3. GREEN — write the minimum code to pass.
4. REFACTOR — tidy up.
5. Repeat until the unit's acceptance criteria are met.

For a bug fix:

1. RED — write a test that reproduces the bug. Confirm it fails on the current code.
2. GREEN — fix the bug. Confirm the test now passes.
3. REFACTOR — improve the surrounding code if useful, keeping tests green.

## Definition of done for a unit

A unit is complete only when:

- All new tests pass.
- All existing tests in the affected files still pass.
- The code is clean (post-refactor), not just functional.

Never declare a unit complete with skipped, disabled, or commented-out tests. If a test cannot be made to pass within the unit's scope, surface it to the orchestrator rather than masking it with `.skip`, `.only`, `xit`, or deletion.

## Testing standards

TDD does not override project conventions. Invoke the `unit-testing-guide` skill and follow its Vitest patterns when writing the failing test — query priority, `must [behavior] when [condition]` naming, `userEvent` over `fireEvent`, flat structure, no snapshot tests, no implementation-detail assertions.
