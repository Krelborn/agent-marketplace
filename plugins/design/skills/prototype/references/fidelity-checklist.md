# High-Fidelity Prototype Checklist

A high-fidelity prototype looks like a finished product screenshot, not a wireframe. Apply this checklist before declaring a prototype complete.

## Visual

- [ ] **Type hierarchy is real.** Headings, body, captions all have distinct sizes and weights. No three sibling elements at the same size.
- [ ] **Spacing is on a scale.** Use the token scale (4/8/12/16/24/32/...). Never `padding: 13px`.
- [ ] **Color has meaning.** Neutrals for surfaces and text; one accent for primary actions; semantic colors only for status (success/warning/danger).
- [ ] **No raw HTML look.** No default browser borders, default `<button>` styling, default `<input>` chrome, or `Times New Roman`.
- [ ] **Consistent radii.** Pick a corner radius and apply it consistently to cards, inputs, buttons.
- [ ] **Shadows are subtle.** One or two elevation levels. Never default `box-shadow`.
- [ ] **Icons are vector.** Inline SVG or icon font (Lucide, Heroicons, Phosphor via CDN). Never emoji as UI icons unless that's the brand.
- [ ] **Imagery is real.** Use Unsplash CDN URLs (`https://images.unsplash.com/...`), realistic avatar generators (`https://i.pravatar.cc/...`), or inline SVG illustrations. No `placeholder.com`.

## Content

- [ ] **Realistic copy.** Real product names, plausible numbers, believable user names. No "Lorem ipsum" except in body-text-as-shape contexts.
- [ ] **Empty states populated.** If showing a list, show 5–10 items with varied data. Don't show one row.
- [ ] **Numbers vary plausibly.** Not "$1,234" repeated 10 times. Mix small and large values, recent and older dates.
- [ ] **States represented.** Show selected, hovered, disabled, error states somewhere if relevant to the flow.

## Layout

- [ ] **Grid feels intentional.** Aligned to a column grid; consistent gutters; nothing accidentally off-axis.
- [ ] **Responsive baseline.** At least one breakpoint at ~768px so it doesn't break on a phone screen. Use `clamp()`, flex, grid — not fixed pixel widths everywhere.
- [ ] **Density is right.** Dashboard density ≠ marketing density. Match the product type.

## Interaction

- [ ] **Hover states.** Buttons, links, cards that look clickable have a hover treatment.
- [ ] **Focus visible.** Keyboard focus is visible (outline ring or border change). Don't `outline: none` without replacement.
- [ ] **At least one working interaction.** A tab switch, a modal open, a toggle, a filter — something that moves. Static screenshots are not prototypes.
- [ ] **Transitions are short.** 150–250ms on hover/state changes. No 1-second animations on hover.

## Code Hygiene (single-file constraint)

- [ ] **One `.html` file, period.** All CSS in `<style>`, all JS in `<script>`. No external `.css` or `.js` files.
- [ ] **External resources are CDN-only.** Fonts (Google Fonts), icons (Lucide CDN, etc.), images (Unsplash, pravatar). No `npm install`, no build step.
- [ ] **Opens with double-click.** No server required. No `file://` issues — avoid `fetch()` of local paths, ES module imports of relative paths.
- [ ] **Reasonable size.** Aim under ~150 KB of HTML. If larger, the prototype is doing too much for one screen.

## What NOT to do

- Don't add React/Vue/Svelte build setups. The output is a single HTML file.
- Don't use Tailwind via CDN — it makes the file bloated and hard to read. Write actual CSS using design tokens.
- Don't apologize in comments ("this is a quick mock"). The prototype should feel finished.
- Don't add fake "TODO" placeholders in the UI. If a feature isn't shown, it isn't shown.
- Don't generate massive prototypes covering 10 screens in one file. One screen per file unless the user asks otherwise; link them with anchor navigation if multiple are needed.
