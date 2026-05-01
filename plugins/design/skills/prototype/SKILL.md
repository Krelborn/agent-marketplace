---
name: prototype
description: This skill should be used when the user asks to "build a prototype", "create a prototype", "make a high-fidelity prototype", "mock up this design", "prototype this sketch", "prototype this wireframe", "turn this spec into a prototype", or otherwise requests a shareable visual mockup of a UI. Produces a single self-contained HTML file with inline CSS and JavaScript so the prototype can be opened in any browser and shared as one file.
---

# Prototype Skill

Generate a single-file, high-fidelity, self-contained HTML prototype from a prompt, written spec, wireframe, or rough sketch. The output is one `.html` file with embedded `<style>` and `<script>` — no build step, no external local assets, opens with a double-click.

## Output Contract

Every prototype produced by this skill MUST satisfy:

1. **One file.** A single `.html` document. CSS lives in one `<style>` block in `<head>`. JS lives in `<script>` blocks. No sibling `.css` / `.js` files.
2. **No build step.** The file opens directly in a browser via `file://`. No `npm`, no bundler, no server required.
3. **External resources are CDN-only and optional.** Google Fonts, icon CDNs (Lucide, Heroicons), image CDNs (Unsplash, pravatar) are allowed. No relative-path imports, no `fetch()` of local files.
4. **High fidelity.** Looks like a finished product screenshot, not a wireframe. See `references/fidelity-checklist.md` for the bar.
5. **At least one working interaction** when the design implies any (tab switch, modal, toggle, filter). Static-only output is acceptable only for purely visual mocks (e.g., a marketing landing hero) where the user explicitly asked for static.

## Workflow

### 1. Read the input

Inputs come as one or more of: prompt text, written spec, image of a wireframe, image of a hand sketch, screenshot of existing UI. For each input type, follow the guidance in `references/input-handling.md`.

Before building, briefly note (internally and in the final response):
- What screen / flow is being prototyped.
- Which input dictates content vs. layout vs. tone (when multiple inputs disagree).
- Any explicit user constraints (brand colors, dark mode required, mobile-only, etc.).

### 2. Decide scope

Default to **one screen per file**. For multiple screens, generate multiple files — one per screen — and name them so the relationship is obvious (`signup-1-email.html`, `signup-2-verify.html`, `signup-3-profile.html`).

The narrow exception: a multi-step flow that the user explicitly wants in a single shareable artifact (e.g., a 3-step onboarding wizard). Then use one file with progressive disclosure (steps shown via tab-like navigation), not anchor scrolling. Even then, ask once before bundling — the default is separate files.

Never stuff unrelated screens into one file. That defeats the share-one-file purpose.

If the request is broad ("prototype the app"), pick the most central screen, build it, and offer the next screen as a follow-up rather than asking for clarification up front.

### 3. Choose the visual direction

Default visual language is the token set in `assets/template.html`: neutral stone palette, Inter font, blue accent, subtle shadows, 8px radius. This produces a modern SaaS look that suits most prototypes.

Override the defaults when:
- The user names a brand, palette, or reference style → match it.
- The project already exists and the prototype must fit in → match the existing style or framework.
- The product type calls for a different feel (consumer mobile app → more saturated, more rounded; finance product → tighter, more conservative; marketing site → larger type scale, more whitespace).
- Dark mode is requested or implied.

Never override defaults arbitrarily. A weirder palette is not better-looking.

### 4. Build the file

Start from `assets/template.html`. The template provides:
- Design tokens (color, type, space, radius, shadow, motion) as CSS variables.
- Light + dark mode via `prefers-color-scheme`.
- A reset and base typography.
- Primitive component classes (`.btn`, `.btn-primary`, `.card`, `.input`).

Read the template, copy its structure into the output file, then:
1. Replace `{{TITLE}}` with a real title.
2. Adjust tokens if the visual direction calls for it (different accent, different radius scale, different font).
3. Add app-specific styles in the marked region — extending the tokens, not redefining them.
4. Build the markup inside `<main>`.
5. Add interaction JS in the bottom `<script>` block.

For common UI patterns (top nav, sidebar, stat cards, data tables, modals, tabs, charts), copy from `references/component-patterns.md` and adapt. Do not hand-roll patterns that already have polished snippets there.

### 5. Populate with realistic content

- Use plausible names, copy, and numbers — not Lorem Ipsum.
- For lists/tables, populate 5–10 rows with varied data.
- For images, use Unsplash CDN URLs with sensible queries (`https://images.unsplash.com/photo-...?w=800`) or `https://i.pravatar.cc/64?u=<seed>` for avatars. Use unique seeds so avatars differ.
- For charts, use inline SVG with hand-tuned paths — see the sparkline pattern in `references/component-patterns.md`. Do not import a chart library.
- For icons, use Lucide via CDN or inline SVG.

### 6. Add the implied interactions

Pick the most obvious working interactions and implement them with vanilla JS:
- A tab/segmented control that actually switches views.
- A modal that actually opens and closes (use `<dialog>`).
- A toggle, switch, or filter that updates visible content.
- Hover/focus states on all interactive elements.

Skip animations beyond ~250ms transitions. The prototype is for review, not for showcasing motion design.

### 7. Self-check against the fidelity bar

Before returning the file, verify against `references/fidelity-checklist.md`. The most common failures:
- Default browser button/input chrome left in place.
- Three sibling elements at the same font size (no hierarchy).
- Real CSS without enough whitespace (cramped).
- One row in a table that should have ten.
- No focus or hover state defined.
- An external `.css` or `.js` file accidentally referenced.

### 8. Save and report

Save the file with a descriptive kebab-case name: `dashboard-prototype.html`, `signup-flow.html`, `pricing-page.html`. Default location is the user's current working directory unless the user specified otherwise.

In the response:
- State the file path.
- Briefly describe what the screen contains and which interactions work.
- Note any explicit assumptions made (brand colors invented, scope reduced from spec, etc.).
- Offer obvious next steps only if there is one (a paired screen, a state variant) — skip the offer otherwise.

## Anti-patterns

- **Tailwind via CDN.** Bloats the file, makes the markup unreadable. Write actual CSS using the token system.
- **React/Vue/Svelte build setups.** The output is a single static HTML file.
- **Multi-file output.** Even if the user wants both desktop and mobile, produce two `.html` files, not a `src/` directory.
- **Imports of local files.** No `<link href="./styles.css">`, no `<script src="./app.js">`, no `import` of relative paths.
- **`placeholder.com` images, `Lorem Ipsum` body copy, "TODO" text in the UI.** Realistic content always.
- **Asking the user to spec every detail.** Make plausible product decisions; the user can redirect on review.
- **Apologizing in code comments** ("// quick mock, ignore"). The prototype should feel finished.

## Resources

- **`assets/template.html`** — Boilerplate single-file scaffold with design tokens, reset, primitives. Start here.
- **`references/fidelity-checklist.md`** — Quality bar to verify against before returning the file.
- **`references/component-patterns.md`** — Polished snippets for nav, sidebar, stat cards, tables, modals, tabs, sparklines, icons.
- **`references/input-handling.md`** — How to translate prompts, specs, wireframes, sketches, and screenshots into a prototype.
