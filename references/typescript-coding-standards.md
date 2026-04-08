# Typescript Coding Standards

Core coding standards for typescript code. These principles MUST be followed when writing or reviewing typescript code.

## Core Principles

1. **Type Safety is Non-Negotiable**: Use TypeScript's type system to its fullest. Prefer strict types over `any` or `unknown` unless `unknown` is genuinely the correct type. Use discriminated unions, generics, mapped types, and conditional types where they add clarity. Every function signature should be an accurate contract.

2. **Performance by Default**: Choose efficient data structures and algorithms. Avoid unnecessary re-renders in React (proper memoization, observer boundaries). Be conscious of bundle size implications. Prefer lazy evaluation and deferred loading where appropriate.

3. **Security-First Mindset**: Sanitize inputs. Validate data at boundaries (especially API responses — use Zod or similar runtime validation). Never trust external data. Avoid XSS vectors. Handle secrets and sensitive data carefully. Use parameterized queries. Escape user content before rendering.

4. **Clarity Over Cleverness**: Write code that a senior developer can understand on first read. Use descriptive variable and function names. Avoid overly terse or "clever" one-liners when a clearer multi-line approach communicates intent better. Code should read like well-written prose.

## Code Style & Standards

### Comments
- Write meaningful comments that explain **why**, not **what** (the code shows what).
- Add JSDoc comments to all public APIs, interfaces, types, and exported functions with `@param`, `@returns`, and tags where applicable.
- Use inline comments sparingly and only when the logic is non-obvious.
- JSDoc comments on interface fields must be multi-line (`/** ... */` on separate lines) with a blank line between each field

### TypeScript Conventions

- Always use curly braces for if/for/while statements, even single-line bodies
- Use camelCase for constants — never SCREAMING_SNAKE_CASE
- Always declare explicit accessibility modifiers (`public`, `private`, `protected`) on class members
- Prefer `interface` over `type` for component props (use `type` only when unions/intersections are needed)
- Member ordering: public fields → private fields → constructor → public methods → protected → private
- Use `readonly` for properties that should not be reassigned after construction
- Prefer `interface` for object shapes that may be extended; use `type` for unions, intersections, and computed types
- Use `const` assertions and `as const` for literal types where appropriate
- Never use `@ts-ignore` — use `@ts-expect-error` with a comment explaining why, only as a last resort
- Prefer `unknown` over `any` when the type is truly not known, and narrow it with type guards
- Always use curly braces for if/for/while statements, even single-line bodies
- Exported functions must have explicit return types (e.g., `: JSX.Element`)

### Function Design

- Functions should do one thing well. If a function exceeds ~30 lines, consider decomposing it
- Prefer pure functions where possible. Isolate side effects
- Use early returns to reduce nesting
- Handle all error paths — never swallow errors silently
- Don't use `async` without `await` — return the promise directly instead

### Error Handling

- Always handle promises — no floating promises
- Use structured error types (custom error classes or discriminated unions) rather than throwing generic `Error` objects
- Catch at appropriate boundaries, not everywhere
- Provide meaningful error messages that aid debugging

### React & MobX Patterns

- Sort props alphabetically in interfaces/types and JSX
- JSX props: alphabetical, then `aria-*`, then `data-*` (each group sorted)
- Sort hook dependency arrays alphabetically
- Destructure React types on import (not `React.ChangeEvent`)
- Use the module pattern: `ComponentName/` folder with `.tsx`, `.module.scss`, `.test.tsx`.
- Clean up disposers and reactions in `useEffect` cleanup functions.
- Wrap React components with `observer()` from `mobx-react-lite` when they read observable state.
- Use `makeObservable` (not `makeAutoObservable`) for classes extending a base class.
- Always explicitly pass boolean values: `disabled={true}` instead of shorthand `disabled`

### CSS Class Names

- Projects may use either `classnames` or `clsx` packages for a utility function for conditional class composition (Never both).
- Prefer `clsx` in new projects
- Projects using "classnames" should `import classNames from "classnames'`
- Projects using "clsx" should `import { clsx } from "clsx";`
- Use class composition utility function for conditional class composition — never ternaries
- Static classes as a single string, conditional classes as an object e.g. `classNames("d-flex flex-grow-1", { "d-none": isHidden })`
- Don't use composition utility when mixing classes without conditionals: `d-flex flext-grow-1 ${styles.fromModule}`

### Import Ordering
- Import ordering: external → `../` internal → `./` local (alphabetical within each group); blank line between each group; type imports before value imports from same module
- Type imports before value imports from the same module

## What You Never Do

- Never use `any` without exhausting all typed alternatives first
- Never leave TODO comments without an explanation of what needs to be done and why it's deferred
- Never ignore error cases or assume happy paths
- Never use `console.log` for error handling in production code
