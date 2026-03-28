# Vertical Slice Guidance

## Core Concept

A vertical slice cuts through ALL layers of the application to deliver a thin piece of complete, working functionality. Each slice proves out end-to-end behavior — from UI to data — and is independently shippable.

```
┌─────────────────────────────────────────┐
│              HORIZONTAL LAYERS           │
├─────────────────────────────────────────┤
│  UI Layer      │ █ │     │     │        │
├────────────────┼───┼─────┼─────┼────────┤
│  API Layer     │ █ │     │     │        │
├────────────────┼───┼─────┼─────┼────────┤
│  Service Layer │ █ │     │     │        │
├────────────────┼───┼─────┼─────┼────────┤
│  Data Layer    │ █ │     │     │        │
└────────────────┴───┴─────┴─────┴────────┘
                  ↑
            Vertical Slice
         (Complete capability)
```

## Vertical vs Horizontal

**Horizontal (avoid):** Build all database models, then all APIs, then all UI, then wire together. Nothing works until everything is done, late integration surprises, hard to show progress.

**Vertical (prefer):** Build one complete user capability at a time. Each slice is shippable, enables continuous integration, and shows visible progress.

## How to Identify Slices

### 1. Start with User Stories

Map each user story to one or more slices. Ask: "What can the user DO after this slice ships?"

### 2. Find the Thinnest Version

For each capability, identify the minimal implementation that proves the behavior works end-to-end:

- Skip validation (add in a follow-up slice)
- Skip edge cases (add in a follow-up slice)
- Skip optimization (add in a follow-up slice)
- Use hardcoded values where full logic isn't needed yet

### 3. Order by Dependency

Which slices enable other slices?

- "View list" before "Filter list"
- "Create item" before "Edit item"
- Core happy path before error handling

### 4. Group Related Refinements

Enhancements to a base slice (validation, edge cases, polish) become sub-slices that reference their parent.

## Slice Sizing

| Size           | Indicators                                                                               |
| -------------- | ---------------------------------------------------------------------------------------- |
| **Too Big**    | Multiple user actions, >2 days work, many acceptance criteria, touches too many concerns |
| **Too Small**  | Just infrastructure, just types, no user-visible behavior, <1 hour work                  |
| **Just Right** | One capability, 1-2 days, 3-5 acceptance criteria, proves one thing end-to-end           |

## Definition of Done

Every slice must have a clear definition of done. Good acceptance criteria are:

- **Observable** — describe what the user sees or can do, not implementation details
- **Testable** — can be verified as pass/fail without ambiguity
- **Independent** — each criterion stands on its own
- **Complete** — together they fully define "done" for this slice

Example:

- "User can see a list of products with name and price" (observable, testable)
- NOT "ProductList component renders correctly" (implementation detail)
- NOT "Works well" (not testable)

## Naming Convention

Use prefixed numbering to show slice relationships:

```
SLICE 1: View empty product list
SLICE 1.1: Display product images in list
SLICE 1.2: Add pagination to product list
SLICE 2: Add new product (happy path)
SLICE 2.1: Add input validation to product form
SLICE 3: Search products by name
```

## Questions to Guide Slicing

1. What is the first thing a user should be able to do? → First slice
2. What is the simplest version of this action? → Strip nice-to-haves
3. What do we need to learn before building more? → Prioritize early slices that reduce uncertainty
4. If we had to ship tomorrow, what would we cut? → Later slices
5. Which slices can be built in parallel by different developers?
