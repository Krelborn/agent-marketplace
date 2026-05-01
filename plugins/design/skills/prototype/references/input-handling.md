# Input Handling

Prototype requests come in different forms. Read the input first, then decide how literally to translate it.

## Prompt-only ("Build me a dashboard for tracking daily steps")

The user gave a feature description with no visuals. Wide latitude on layout and styling.

**Approach:**
1. Identify the **primary screen** — the one the user clearly cares about. If they mention multiple, ask which to prototype first or pick the most central one.
2. Identify the **key data** the screen displays (3–6 elements max for a single-page proto).
3. Identify the **primary action(s)** — what the user does on this screen.
4. Choose a layout pattern that fits the product type:
   - **Dashboard / analytics** → top nav + sidebar + grid of stat cards + 1–2 charts + a table.
   - **List / CRM** → top nav + filter bar + table or card grid + detail drawer.
   - **Marketing site** → hero + features grid + social proof + CTA.
   - **Settings / form** → centered single column, sectioned, sticky save bar.
   - **Mobile-first** → constrain max-width to ~420px, stacked layout.
5. Make plausible product decisions (a fitness tracker shows steps, calories, distance, weekly trend) without asking the user to spec every field.

## Spec / requirements doc

The user provided a written spec listing screens, features, fields, copy.

**Approach:**
1. Treat the spec as authoritative for **content and structure** (field names, labels, sections, copy).
2. Treat it as advisory for **visual hierarchy** (the spec usually doesn't say "make this the largest element").
3. If the spec lists more than fits on one screen, prototype the most important slice. Note in the response what was scoped out.
4. Don't invent fields that aren't in the spec; don't drop fields that are unless space forces it (then mention).

## Wireframe (low-fi sketch, boxes-and-lines)

The user provided an image of a wireframe — gray boxes, "Lorem" placeholders, text labels for sections.

**Approach:**
1. **Preserve the layout topology**: header where they put the header, sidebar where they put the sidebar, content regions in the same relative positions and proportions.
2. **Upgrade fidelity**: real type, real spacing, real components. The wireframe shows "image goes here" — replace with an actual Unsplash image. It shows "button" — render a styled button with realistic label.
3. **Don't reinvent the structure.** If the wireframe puts the search bar on the left and the user avatar on the right, that's the layout.
4. **Fill in plausible content** based on context cues (a "products" wireframe gets product cards with realistic names, prices, images).

## Rough sketch (hand-drawn, photographed)

The user uploaded a photo of a hand sketch — possibly imprecise, possibly hard to read.

**Approach:**
1. **Identify the structural intent**, not the literal pixels. Hand sketches are about regions and flow, not measurements.
2. **List what is clearly intended** before building (e.g., "I see: a header with logo + nav, a hero section with headline and CTA, a 3-column features grid below"). If anything is ambiguous, note the assumption rather than asking — the user can correct on review.
3. **Do not preserve sketch artifacts**: jaggy lines, scribbled text. Render as a clean modern UI.
4. **Hand-drawn marginal annotations** ("user can filter here") are content cues — implement the feature, don't render the annotation.

## Existing design / screenshot

The user provided a screenshot of an existing UI and wants something "like this" or wants it remade.

**Approach:**
1. Match the **layout, density, and overall aesthetic** (light/dark, rounded/sharp, dense/spacious).
2. Don't pixel-clone — extract the design language and apply it to the user's actual content.
3. If the user said "but for X product instead of Y", change content, keep structure.

## Multiple inputs (e.g., spec + sketch)

When user provides several artifacts: the **spec wins on content**, the **sketch wins on layout**, the **prompt wins on tone/scope**. Reconcile contradictions explicitly in the response rather than picking silently.

## When to ask before building

Default to building. Ask only if:
- The input is genuinely undecodable (sketch is illegible, spec is contradictory in a way that changes the structure).
- The scope is ambiguous to a degree that would mean building the wrong thing entirely (e.g., "build the app" with no hint of which screen).
- Brand/visual constraints exist that aren't specified and matter (e.g., user mentioned a brand but didn't share colors).

Never ask about minor styling choices, copy, or sample data — those are judgment calls the prototype is supposed to make.
