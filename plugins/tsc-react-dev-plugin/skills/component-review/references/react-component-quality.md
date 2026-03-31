# React Component Quality Principles

## 1. DRY Wrappers and Composition

**Multiple components sharing the same wrapper JSX with identical styles is a duplication smell.**

- If 2+ components wrap content in the same container structure (e.g., `<div className="card p-4 rounded shadow">`), extract a shared wrapper component.
- If components differ only in children but share layout/styling, extract the shared layout as a composition component that accepts `children`.
- Wrapper components should accept `className` to allow consumers to extend styles.

```tsx
// Bad — duplicated in ProfileCard, SettingsCard, NotificationCard
const ProfileCard = () => (
  <div className="card p-4 rounded shadow">
    <h2>Profile</h2>
    ...
  </div>
);

// Good — shared wrapper
const Card = ({ children, className }: CardProps) => (
  <div className={clsx("card p-4 rounded shadow", className)}>{children}</div>
);

const ProfileCard = () => (
  <Card>
    <h2>Profile</h2>
    ...
  </Card>
);
```

Also watch for:

- Repeated icon+text label patterns — extract a `LabeledIcon` or similar
- Repeated list rendering with identical item wrappers — extract the item component
- Repeated modal/dialog structures — extract a base modal component

## 2. Props Documentation

**Reusable components must have documented props.**

A component is "reusable" if any of:

- Exported from its module
- Used in 2+ places
- Designed as a shared UI primitive (lives in a shared/common directory)

Documentation requirements:

- Props interface/type must have TSDoc comments on non-obvious fields
- Required vs optional should be clear from the types (no `prop?: Type` when it's actually required)
- Callback props should document when they fire and what the arguments represent
- Generic components should document type parameter constraints

Internal, single-use components (only used by their parent) don't need props docs.

## 3. Simplicity

**Favour the simplest implementation that meets requirements.**

Flag:

- Unnecessary abstraction layers (wrapper components that just pass props through without adding value)
- Premature generalization (making a component configurable for use cases that don't exist yet)
- Overly clever patterns (render props or HOCs where a simple component with children would work)
- Components doing too many things (mixing data fetching, business logic, and presentation)
- Deeply nested ternaries in JSX — extract to variables or early returns
- Excessive `useMemo`/`useCallback` without evidence of performance need

A component should have one clear responsibility. If describing what a component does requires "and", it may need splitting.

## 4. Component Structure

**Components should be predictable and consistently structured.**

- Hooks at the top, in a consistent order: state, context, refs, effects, callbacks, derived values
- Early returns for loading/error/empty states before the main render
- Event handlers defined before the return statement, not inline (unless trivial one-liners)
- Avoid deeply nested JSX (4+ levels of indentation usually means extract a sub-component)
- Conditional rendering: prefer early returns or separate components over complex inline conditions

## 5. Accessibility

**Components must be usable by all users.**

- Interactive elements must be keyboard accessible (buttons, links, custom controls)
- Semantic HTML: use `<button>` not `<div onClick>`, `<nav>` not `<div className="nav">`, `<ul>/<li>` for lists
- Images need `alt` text (empty `alt=""` for decorative images)
- Form inputs need associated labels (via `<label htmlFor>` or `aria-label`)
- Custom interactive components need appropriate ARIA roles and states
- Colour must not be the only means of conveying information
- Focus management: modals should trap focus, closing should return focus

Don't flag every missing aria attribute — focus on interactive elements and content that would be unusable without accessibility support.

## 6. Prop Drilling

**Avoid passing props through multiple intermediate components.**

If a prop passes through 3+ levels without being used by intermediaries:

- Consider React Context for app-wide or subtree-wide state
- Consider component composition (passing components as props/children instead of data)
- Consider moving the consuming component closer to the data source

Not every prop pass is "drilling" — 2 levels is normal. Flag it when intermediaries are pure pass-throughs with no use for the prop.

## 7. Controlled vs Uncontrolled Consistency

**Components should be clearly one or the other, not a confusing mix.**

- A controlled component receives its value and onChange from the parent
- An uncontrolled component manages its own internal state
- A component that accepts both `value` and `defaultValue` must handle them correctly (controlled takes precedence)
- Flag components that accept `value` but also maintain internal state that can diverge

## 8. Key Prop Usage

**Lists must use stable, unique keys — never array indices (unless the list is static and never reordered).**

- Keys should come from the data (IDs, slugs, unique identifiers)
- Flag `key={index}` on dynamic lists
- Flag missing keys on mapped elements
- Flag keys that aren't unique within their sibling set

## 9. Error Boundaries and Resilience

**Components should fail gracefully.**

- Components that fetch data or render dynamic content should handle error states
- Shared/reusable components that render children should consider an error boundary
- Don't swallow errors silently — show meaningful feedback to the user
- Optional chaining on data that could be undefined is preferred over crashing

## 10. Performance Awareness

**Avoid obvious performance anti-patterns, but don't over-optimize.**

Flag:

- Creating new objects/arrays/functions in render that are passed as props to memoized children (breaks memo)
- Expensive computations in render without `useMemo`
- Missing `observer()` wrapper on components reading MobX observables
- Large components that re-render entirely when only a small part changes (consider splitting)

Don't flag:

- Missing `React.memo` on every component (only flag when there's a clear performance impact)
- Missing `useCallback` on handlers that aren't passed to memoized children
