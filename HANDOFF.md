# Signifyd Website — Build Handoff

Static site: `index.html` + `styles.css` at repo root, `assets/` for media. No build step — open `index.html` directly or serve the folder. External deps loaded via CDN: **Lenis** (smooth scroll) and **Swiper** (carousel).

This doc exists so you don't have to reverse-engineer the CSS from scratch. Read the "Design system" section before touching any styling — almost every value on the site should come from an existing variable, not a new hardcoded one.

## Design system — everything lives in `:root` (top of `styles.css`)

The whole theme was pulled 1:1 from a Figma variables collection, so variable names below match Figma's own names. **Rule: no real color, font size, or grid width should ever be hardcoded inline — always read from a variable.**

### Colors

| Variable | Value | Use |
|---|---|---|
| `--color-text` / `--color-headline` | `#0d0d0d` | body text / headlines |
| `--color-background` | `#fefdfb` | page background |
| `--color-background-invert` | `#1e1e1e` | dark section background |
| `--color-background-lighter` / `--color-background-light` | `#f8f8f8` / `#efefef` | subtle surface tints |
| `--color-brand` | `#d5ff02` | lime accent |
| `--color-cta-text` / `--color-cta-text-inverted` | `#fefdfb` / `#0d0d0d` | text on filled vs. light buttons |
| `--color-cta-secondary` | `#f4f3f1` | secondary button fill |
| `--color-cta-hover-2` | `rgba(13,13,13,.1)` | button hover wash (fades in via `opacity`, not by animating this alpha) |
| `--color-grid-line` | `rgba(13,13,13,.2)` | the dashed grid lines used everywhere |
| `--color-status-verified-purple` / `-lime` / `-danger` | — | decision-card mockup accents (not in Figma's theme, added for the platform section) |

**Theme indirection (important pattern):** components never read `--color-*` directly for bg/ink/button — they read `--theme-bg`, `--theme-ink`, `--theme-btn-primary-bg`, `--theme-btn-primary-text`, `--theme-grid-line`. Adding `.theme-inverted` to a section wrapper re-points all five at once to the dark variants. Used on Platform and Big Numbers sections. **If you add a new dark section, apply `.theme-inverted` to its wrapper rather than writing new dark-mode CSS.**

### Typography

Two families only:
- **Inter** — all body copy, headlines, nav, buttons.
- **Fragment Mono** — small uppercase labels only (eyebrows, card index tags, tab labels, footer tagline).

Font-size scale (frozen to Figma's fixed 1440px canvas — intentionally **no fluid/clamp scaling**):

| Variable | px | Used for |
|---|---|---|
| `--font-size-2xs` | 14 | mono labels, index tags |
| `--font-size-xs` | 15 | eyebrows, CTA label, footer titles |
| `--font-size-sm` | 16 | nav, links, body copy |
| `--font-size-md` | 18 | subheads/intros, card titles |
| `--font-size-lg` | 22 | hero subhead |
| `--font-size-xl` | 28 | stats feature title |
| `--font-size-2xl` | 32 | solutions panel heading |
| `--font-size-3xl` | 42 | section headlines (every section but hero) |
| `--font-size-4xl` | 54 | big stat numbers |
| `--font-size-5xl` | 66 | hero headline |

Font-weights in use: 400 (body), 500, 600, 700 (headlines/emphasis) — no other weights. Line-heights cluster around `1` (tight numerals/labels), `1.1` (headlines), `1.4–1.5` (body/paragraphs) — reuse those rather than picking a new value.

### Grid & layout

- `--grid-max-width: 1256px` + `--grid-padding: 32px` — every section's content is capped and centered via the `.grid-frame` utility class. Use `.grid-frame` on any new section's inner content wrapper rather than writing new max-width/padding rules.
- `--canvas-width: 1440px` — Figma's own canvas width. Any decorative content sized as a % (floating cards, background art, hero video) needs its own wrapper capped to this, or it drifts on screens wider than 1440px.
- `.grid-frame::before` draws the dashed vertical grid lines (a repeating-gradient pseudo-element, z-index 2, pointer-events none) — don't add grid lines manually elsewhere, extend this pattern.
- Inner content grids are plain CSS Grid: `repeat(12, 1fr)` for most content rows, `repeat(5, minmax(0,1fr))` for the logo/trust bar. Collapses to `1fr` (stacked) or `repeat(2, minmax(0,1fr))` at the section's breakpoint.
- Breakpoints in use (all `max-width`, no `min-width` queries): **1024px, 900px, 768px, 720px, 640px, 600px, 560px**. Most sections use 900px as their primary "stack to mobile" break — match that unless a section has its own reason not to (check what's already there before adding a new breakpoint value).

### Motion / other tokens

- Border-radius: 4px and 8px cover almost everything; 999/9999px for pills; a couple of one-off 6px/14px.
- Transitions: `opacity` fades use plain `ease`; anything that slides/pushes (image crossfades, tab indicator) uses `cubic-bezier(0.65, 0, 0.35, 1)` — reuse that curve for any new push/slide motion so it feels consistent.

## Section inventory (top to bottom in `index.html`)

| Wrapper class | Status |
|---|---|
| `.hero` | live |
| `.social-proof-wrapper` | live |
| `.pain-points-wrapper` | live |
| `.platform-wrapper` | live, `.theme-inverted` |
| `.solutions-wrapper` | live — "Commerce begins before the cart" scrollytelling section. Sticky-centers via pure flexbox (no JS measurement for centering), copy scrolls through a `mask-image`-cropped window in one continuous motion, image uses a push-crossfade transition. Tunable via custom properties on `.solutions-wrapper` itself (see below). |
| `.testimonials-block` | live |
| `.stats-wrapper` | live |
| `.big-numbers-block` | live, `.theme-inverted` |
| `.copyrights-wrapper` | live (footer) |

Housekeeping note: this used to have three retired experiments living alongside it (`solutions-alt`, `solutions-alt2`, and a "`solutions4`" rebuild) — all `display:none`, kept only for history. They've been removed along with their CSS/JS, and the winning version's classes/variables were renamed from the `solutions4-`/`--s4-` scratch prefix to the plain `solutions-`/`--solutions-` names shown above, since it's the only version now.

### `.solutions-wrapper` tunable knobs (custom properties on the wrapper itself)

| Variable | Value | Controls |
|---|---|---|
| `--solutions-buffer` | 48px | total gap above the alignment line (hard part + fade) |
| `--solutions-top-fade` | 44px | of that gap, how much is the actual fade gradient |
| `--solutions-fade-drop` | 24px | pushes the top fade down past the line, so copy at the line is still fading |
| `--solutions-window` | 502px | fixed masked-window height (whole inner block renders ~700px) |
| `--solutions-bottom-fade` | 56px | soft bottom edge of the copy window |
| `--solutions-gap` | 24vh | spacing between topics along the copy track |
| `--solutions-panels` | 4 | number of topics — drives the scroller height |
| `--solutions-runway-vh` | 10 | dwell/release runway after the last topic (in vh) |
| `--solutions-pad` | 56px | vertical breathing room inside the pinned stage |

These were tuned by eye in-browser already — if you're adjusting the "fully opaque exactly at the alignment line" feel, start here rather than the underlying `mask-image` rules.

## Key technical patterns to know before editing

- **Smooth scroll is global via Lenis** — any custom scroll-driven JS (like the solutions copy scroller) reads scroll progress off Lenis-driven values, not raw `window.scrollY`, and any programmatic scroll (tab clicks) goes through `lenis.scrollTo(...)`, not `window.scrollTo`.
- **Active-state tracking uses `IntersectionObserver`**, not scroll-position math — e.g. the solutions section's tab sync watches a line at the vertical center of its copy window (`root: copyWindow`, `rootMargin: '-50% 0px -50% 0px'`) and whichever heading crosses it wins. This pattern is deliberately reused because it stays correct even when the element it's observing moves via CSS `transform` (raw scroll-offset math would not).
- **Image crossfades are transform-based, not opacity crossfades**: `--active` → `translateY(0)`, `--offscreen-top` → `translateY(-100%)`, base state → `translateY(100%)`, `transition: transform 0.6s cubic-bezier(0.65,0,0.35,1)`. This exact mechanism is used everywhere a "changing visual" appears (solutions section, hero's rotating word pill) — don't reinvent it for a new one.
- **Nested custom properties are avoided on purpose** for anything read across theme swaps (`--theme-grid-line`, hover washes) — the CSS comments call this out explicitly: nesting bakes in whatever value was active at the *declaration* site, not the *usage* site, which silently breaks theming. Follow the existing pattern (read the base token directly at point of use) rather than "cleaning up" by nesting one custom property inside another.

## Open items / things flagged but not resolved

- The `.solutions-wrapper` tunable values above have already had one visual tuning pass; if timing/positioning ever feels off on a very short or very tall viewport, that's the first place to check.
- Confirm `.solutions-wrapper` releases cleanly into `.testimonials-block` with no dead scroll gap, at multiple viewport heights.
- There's still a `TEMP DEBUG` parallax-tuning panel in `.platform-wrapper` (`styles.css`, search "TEMP DEBUG") toggled via Shift+S — left in on purpose per its own comment ("remove once platform parallax values are finalized"). Worth checking with whoever owns that section whether it's safe to remove now.
- Two content additions were proposed but not built during a separate page-design exploration (a Figma wireframing pass on a Purchase Optimization landing page, not this repo directly, but relevant if this site grows a product-page track): a short "a guarantee alone isn't enough" comparison callout, and an FAQ block before the closing CTA.

## Repo/history reference

Recent commits (`git log`) trace the current state: hero visual narrowed, pain-point/solution imagery updated to v2 assets, an `solutions-alt2` variant built as an A/B candidate then retired in favor of what's now the live `.solutions-wrapper`, scrollspy reworked twice to fix Lenis race conditions and to sync against the reading line instead of viewport center. The retired experiments and their scratch class/variable prefixes have since been deleted and renamed — if history looks confusing around the Solutions section, that's why. If something looks intentional-but-odd elsewhere, check `git log -p` on that file region before "fixing" it.
