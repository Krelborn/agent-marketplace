# Visual Design Guidance

Reference this file when producing visual design artifacts during the Visual Design Phase.

## HTML Mockup Creation

Create mockups as single self-contained `.html` files that open directly in a browser.

### Structure

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>[Feature] - Visual Mockup</title>
    <style>
      /* Design tokens as CSS custom properties */
      :root {
        --color-primary: #...;
        --color-neutral-900: #...;
        --font-body: "...", sans-serif;
        --space-md: 16px;
        --radius-md: 8px;
        /* ... */
      }
      /* Component styles */
    </style>
  </head>
  <body>
    <!-- Mockup content -->
  </body>
</html>
```

### Rules

- **No external dependencies** - all CSS inline, no CDN links (exception: Google Fonts `<link>` for typography)
- **CSS custom properties** for all design tokens - these map directly to the Design System Specification
- **Responsive** via media queries - show the design works at mobile, tablet, and desktop
- **Realistic content** - use plausible text and data, not lorem ipsum
- **Key states** - show default, empty, loading, and error states where relevant (use separate sections or CSS classes to toggle)
- **Minimal JavaScript** - only for state toggling or tab switching within the mockup, not for business logic

### What to include

- Layout and spatial relationships
- Color application (backgrounds, text, borders, accents)
- Typography hierarchy (headings, body, labels, metadata)
- Component appearance in key states (hover via CSS `:hover`, disabled via class)
- Responsive behavior at defined breakpoints
- Realistic placeholder content

### What to skip

- Real API calls or data fetching
- Complex interactivity (drag-and-drop, real form submission)
- Animation (unless core to the design - then use CSS transitions)
- Production-quality code structure

## Design Principles

- **Be deliberate** - every color, font, and spacing choice should have a reason. Avoid falling back to safe defaults (Inter, #6366f1, 8px radius on everything)
- **Match the context** - a finance dashboard looks different from a social app. Let the product's domain and audience drive the aesthetic
- **Establish hierarchy** - use size, weight, color, and spacing to make the most important elements stand out
- **Consistency over novelty** - within a design, repeat patterns. Use the same spacing scale, the same border radius, the same shadow depth
- **Accessibility first** - ensure 4.5:1 contrast ratio for text, visible focus indicators, sufficient touch targets (44x44px minimum)

## Design Token Format

Define tokens in the Design System Specification document and mirror them as CSS custom properties in mockups:

```
Colors:     --color-{category}-{shade}     e.g., --color-primary, --color-neutral-700
Typography: --font-{usage}                 e.g., --font-heading-lg, --font-body
Spacing:    --space-{size}                 e.g., --space-sm, --space-lg
Shape:      --radius-{size}, --shadow-{size}
```

This naming convention ensures the design system doc and HTML mockups stay in sync and translate cleanly into implementation.

## Responsive Strategy

Define three breakpoints consistently across all mockups:

| Breakpoint | Width      | Target                   |
| ---------- | ---------- | ------------------------ |
| Mobile     | < 768px    | Phone portrait           |
| Tablet     | 768-1024px | Tablet / phone landscape |
| Desktop    | > 1024px   | Laptop and above         |

Use mobile-first CSS (`min-width` media queries). Document what changes at each breakpoint:

- Layout shifts (single column to multi-column)
- Navigation changes (hamburger to full nav)
- Component adaptations (stacked to side-by-side)
- Content visibility (show/hide secondary content)
