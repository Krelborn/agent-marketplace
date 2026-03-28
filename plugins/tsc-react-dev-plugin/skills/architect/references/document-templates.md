# Document Templates

## Table of Contents

- [Requirements Document](#requirements-document)
- [Component Design Document](#component-design-document)
- [Technical Design Document](#technical-design-document)
- [API Contract Document](#api-contract-document)
- [Task Breakdown Document](#task-breakdown-document)
- [State Management Design](#state-management-design)

---

## Requirements Document

Use this template when capturing functional and non-functional requirements for a feature or component.

```markdown
# Requirements: [Feature/Component Name]

## Problem Statement

[What problem does this solve? Who experiences it? What is the impact?]

## User Stories

- As a [role], I want [capability] so that [benefit]
- ...

## Functional Requirements

### FR-1: [Requirement Name]

- **Priority:** Must Have | Should Have | Nice to Have
- **Description:** [Clear description of the requirement]
- **Acceptance Criteria:**
  - [ ] [Testable criterion]
  - [ ] [Testable criterion]

## Non-Functional Requirements

### NFR-1: Performance

- [Specific, measurable performance targets]

### NFR-2: Accessibility

- [WCAG level, specific a11y requirements]

### NFR-3: Browser/Device Support

- [Supported browsers, viewport ranges]

## Constraints

- [Technical, business, or timeline constraints]

## Dependencies

- [External APIs, libraries, other components]

## Out of Scope

- [Explicitly excluded items]

## Open Questions

- [ ] [Unresolved question needing stakeholder input]
```

---

## Component Design Document

Use this template when designing React components - their structure, props, behavior, and composition.

```markdown
# Component Design: [Component Name]

## Overview

[Brief description of purpose and where it fits in the UI]

## Component Hierarchy
```

ParentComponent
├── ComponentA
│ ├── SubComponentA1
│ └── SubComponentA2
└── ComponentB

```

## Props Interface

| Prop | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| name | string | Yes | - | Description |

## Component States
- **Default:** [Description]
- **Loading:** [Description]
- **Error:** [Description]
- **Empty:** [Description]

## User Interactions
1. [Action] -> [Result]
2. [Action] -> [Result]

## Accessibility
- Keyboard navigation: [Details]
- Screen reader: [ARIA attributes, roles]
- Focus management: [Details]

## Responsive Behavior
- **Mobile (< 768px):** [Layout description]
- **Tablet (768-1024px):** [Layout description]
- **Desktop (> 1024px):** [Layout description]

## Edge Cases
- [Edge case and expected behavior]

## Dependencies
- [Libraries, hooks, context providers]
```

---

## Technical Design Document

Use this template for broader technical designs spanning multiple components or system-level concerns.

```markdown
# Technical Design: [Feature Name]

## Summary

[1-2 paragraph overview of the technical approach]

## Goals

- [Goal 1]
- [Goal 2]

## Non-Goals

- [Explicitly excluded from this design]

## Architecture

### System Diagram

[Describe data flow and component relationships]

### Key Decisions

| Decision   | Options Considered | Choice | Rationale |
| ---------- | ------------------ | ------ | --------- |
| [Decision] | A, B, C            | B      | [Why]     |

## Data Model

[TypeScript interfaces, data shapes, or schema descriptions]

## State Management

- **Client state:** [What, where, why]
- **Server state:** [Caching strategy, sync approach]
- **URL state:** [Query params, route state]

## API Integration

- [Endpoints consumed, request/response shapes]

## Error Handling Strategy

- [How errors surface to users, retry logic, fallbacks]

## Performance Considerations

- [Bundle size impact, render optimization, lazy loading]

## Security Considerations

- [XSS prevention, input sanitization, auth concerns]

## Migration / Rollout Plan

- [Phased approach, feature flags, backward compatibility]

## Risks and Mitigations

| Risk   | Likelihood   | Impact       | Mitigation |
| ------ | ------------ | ------------ | ---------- |
| [Risk] | High/Med/Low | High/Med/Low | [Plan]     |

## Open Questions

- [ ] [Question]
```

---

## API Contract Document

Use this template when defining the interface between frontend and backend.

````markdown
# API Contract: [Feature/Endpoint Group]

## Overview

[What this API supports in the frontend]

## Endpoints

### [METHOD] /api/path

**Purpose:** [What this endpoint does]

**Request:**

```typescript
interface RequestBody {
  field: string;
}
```
````

**Response (200):**

```typescript
interface ResponseBody {
  data: SomeType;
}
```

**Error Responses:**
| Status | Code | Description |
|--------|------|-------------|
| 400 | VALIDATION_ERROR | [When this occurs] |
| 404 | NOT_FOUND | [When this occurs] |

**Frontend Usage:**

- [Which component/feature consumes this]
- [Caching/polling strategy]

## Shared Types

[TypeScript types shared between request/response]

## Authentication

[Auth requirements for these endpoints]

## Rate Limiting / Pagination

[Relevant constraints]

````

---

## Design System Specification

Use this template when establishing or documenting a project's visual language and design tokens.

```markdown
# Design System: [Project/Product Name]

## Visual Tone

[1-2 sentences describing the intended personality: e.g., "Professional and calm with moments of warmth" or "Bold and energetic with a playful edge"]

## Color Palette

### Primary
| Token | Hex | Usage |
|-------|-----|-------|
| --color-primary | #XXXXXX | Primary actions, key UI elements |
| --color-primary-light | #XXXXXX | Hover states, backgrounds |
| --color-primary-dark | #XXXXXX | Active states, emphasis |

### Neutral
| Token | Hex | Usage |
|-------|-----|-------|
| --color-neutral-900 | #XXXXXX | Primary text |
| --color-neutral-700 | #XXXXXX | Secondary text |
| --color-neutral-400 | #XXXXXX | Placeholder text, borders |
| --color-neutral-200 | #XXXXXX | Dividers, subtle borders |
| --color-neutral-50 | #XXXXXX | Backgrounds |

### Semantic
| Token | Hex | Usage |
|-------|-----|-------|
| --color-success | #XXXXXX | Success states, confirmations |
| --color-warning | #XXXXXX | Warnings, caution |
| --color-error | #XXXXXX | Errors, destructive actions |
| --color-info | #XXXXXX | Informational states |

## Typography

| Token | Font | Size | Weight | Line Height | Usage |
|-------|------|------|--------|-------------|-------|
| --font-heading-xl | [Family] | [Size] | [Weight] | [LH] | Page titles |
| --font-heading-lg | [Family] | [Size] | [Weight] | [LH] | Section headings |
| --font-heading-md | [Family] | [Size] | [Weight] | [LH] | Card titles |
| --font-body | [Family] | [Size] | [Weight] | [LH] | Body text |
| --font-body-sm | [Family] | [Size] | [Weight] | [LH] | Captions, helper text |
| --font-label | [Family] | [Size] | [Weight] | [LH] | Labels, tags |

## Spacing Scale

| Token | Value | Usage |
|-------|-------|-------|
| --space-xs | 4px | Tight gaps, inline spacing |
| --space-sm | 8px | Related element gaps |
| --space-md | 16px | Standard padding, section gaps |
| --space-lg | 24px | Card padding, group separation |
| --space-xl | 32px | Section separation |
| --space-2xl | 48px | Page-level spacing |

## Shape and Elevation

| Token | Value | Usage |
|-------|-------|-------|
| --radius-sm | [Value] | Buttons, inputs |
| --radius-md | [Value] | Cards, panels |
| --radius-lg | [Value] | Modals, sheets |
| --shadow-sm | [Value] | Subtle elevation (dropdowns) |
| --shadow-md | [Value] | Cards, raised elements |
| --shadow-lg | [Value] | Modals, popovers |

## Motion

- **Duration:** [e.g., 150ms for micro-interactions, 300ms for transitions, 500ms for entrances]
- **Easing:** [e.g., ease-out for entrances, ease-in for exits, ease-in-out for state changes]
- **Principles:** [e.g., "Subtle and purposeful - motion should guide attention, not distract"]

## Component Patterns

Brief visual description of core component treatments:

- **Buttons:** [e.g., "Rounded with solid fill for primary, outlined for secondary, subtle hover lift"]
- **Inputs:** [e.g., "Bottom-border style with floating labels, focus ring in primary color"]
- **Cards:** [e.g., "Rounded corners, subtle shadow, white background, hover shadow increase"]
- **Navigation:** [e.g., "Fixed top bar with logo left, nav center, actions right"]
```

---

## Visual Design Specification

Use this template when defining the visual design for a specific feature or screen. Pairs with HTML mockup files.

```markdown
# Visual Design: [Feature/Screen Name]

## Overview

[What this screen/feature looks like and its role in the user experience]

## HTML Mockup

[Link to mockup file, e.g., `mockups/feature-name.html` - open in browser to preview]

## Layout

### Desktop (> 1024px)
[Describe layout structure: grid areas, sidebar placement, content flow]

### Tablet (768-1024px)
[Describe layout changes: collapsed elements, reflow behavior]

### Mobile (< 768px)
[Describe mobile layout: stacked elements, hidden navigation, bottom sheets]

## Component Visual States

| Component | Default | Hover | Active | Disabled | Focus | Error |
|-----------|---------|-------|--------|----------|-------|-------|
| [Component] | [Description] | [Description] | [Description] | [Description] | [Description] | [Description] |

## Color Usage

- **Background:** [Token and rationale]
- **Primary action:** [Token and rationale]
- **Text hierarchy:** [Which tokens for headings, body, secondary text]
- **Accent/highlight:** [Where and why]

## Typography Usage

- **Page title:** [Token]
- **Section headings:** [Token]
- **Body content:** [Token]
- **Labels/metadata:** [Token]

## Spacing and Alignment

- **Content max-width:** [Value]
- **Grid columns:** [Value and gutter]
- **Section spacing:** [Token]
- **Component internal padding:** [Token]

## Interactions and Animation

- [Interaction]: [Visual behavior, duration, easing]
- [Interaction]: [Visual behavior, duration, easing]

## Design Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| [Decision] | [Choice] | [Why this serves the user/brand] |

## Open Visual Questions

- [ ] [Unresolved visual decision needing input]
```

---

## Task Breakdown Document

Use this template when breaking a design into vertical slices for implementation. Each slice delivers end-to-end functionality that proves out a user capability.

```markdown
# Task Breakdown: [Feature Name]

## Source Documents

- Requirements: [link or reference]
- Technical Design: [link or reference]
- Visual Design: [link or reference, if applicable]

## Stories Covered

[List the user stories from the requirements doc that this breakdown delivers]

## Vertical Slices

### SLICE 1: [Capability Name]

**Story:** As a [role], I want [capability] so that [benefit]

**Layers touched:** [UI, API, Service, Data — whichever apply]

**Description:** [1-2 sentences on what this slice proves end-to-end]

**Definition of Done:**
- [ ] [Observable, testable acceptance criterion]
- [ ] [Observable, testable acceptance criterion]
- [ ] [Observable, testable acceptance criterion]

**Dependencies:** None (first slice) | SLICE N

**Risk/Uncertainty:** Low | Medium | High — [brief reason if Medium/High]

**Notes:** [Anything the implementer should know — shortcuts taken, what is deliberately deferred]

### SLICE 1.1: [Refinement of Slice 1]

**Story:** [Same or related story]

**Layers touched:** [Layers]

**Description:** [What this adds to the base slice]

**Definition of Done:**
- [ ] [Criterion]

**Dependencies:** SLICE 1

### SLICE 2: [Next Capability]

...

## Slice Dependency Graph

[Show ordering and parallelism. Text or ASCII diagram.]

```
SLICE 1 ──→ SLICE 1.1
         ──→ SLICE 1.2
SLICE 2 ──→ SLICE 3
SLICE 1 + SLICE 2 can be parallelized
```

## Summary

| Slice | Capability | Dependencies | Risk | Est. Size |
|-------|-----------|-------------|------|-----------|
| 1 | [Name] | None | Low | [1-2 days] |
| 1.1 | [Name] | Slice 1 | Low | [0.5-1 day] |
| 2 | [Name] | None | Medium | [1-2 days] |

## Open Questions

- [ ] [Anything unresolved that affects slicing]
```

---

## State Management Design

Use this template when designing complex state management for a feature.

```markdown
# State Management Design: [Feature Name]

## State Overview
[What state this feature manages and why]

## State Categories

### Server State
| Data | Source | Cache Strategy | Invalidation |
|------|--------|---------------|--------------|
| [Data] | [API endpoint] | [stale-while-revalidate, etc.] | [When] |

### Client State
| State | Scope | Location | Reason |
|-------|-------|----------|--------|
| [State] | [Component/Feature/Global] | [useState/context/store] | [Why here] |

### URL State
| Param | Type | Purpose |
|-------|------|---------|
| [param] | string | [What it controls] |

## State Flow Diagram
[Describe how state flows between components]

## Optimistic Updates
- [Which actions use optimistic updates and rollback strategy]

## Error State Handling
- [How each type of error is captured and surfaced]
````
