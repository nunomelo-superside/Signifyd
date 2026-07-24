# Brief: rebuild the "Solutions" section from scratch as `solutions4`

## Files
- `index.html` and `styles.css` at the repo root (the "Signifyd" project).
- Do not touch or extend `.solutions-alt2-*` (see "Why not alt2" below) — build an
  entirely new, independent section with its own class prefix: `solutions4-`.
- Reference (but do not modify) the **original hidden section**, `.solutions-wrapper`,
  currently sitting in `index.html` with `style="display:none" data-hidden-for-debug="true"`
  right before the `.solutions-alt-wrapper` block. Its image crossfade technique and
  overall proportions are proven and should be reused.
- There are three existing variants in the file today, all doing the same
  "Commerce begins before the cart" section: `.solutions-wrapper` (original, hidden),
  `.solutions-alt-wrapper` (hidden), `.solutions-alt2-wrapper` (currently the live one).
  Once `solutions4` is built and approved, set `.solutions-alt2-wrapper` to
  `style="display:none" data-hidden-for-debug="true"` (matching the other two) so
  `solutions4` becomes the one that actually renders.

## Why not alt2
Alt2 works but got there through a long chain of JS-computed measurements that
depend on each other (title/subtitle height → padding-top for centering →
tabs/visual `top` offset → rail heights → a separately-positioned fade overlay's
`top` → negative margins to cancel grid-row auto-sizing). It's fragile: change any
copy or font size and several unrelated numbers have to re-derive correctly together.
The client doesn't trust it and wants a rebuild with far fewer moving, interdependent
parts — ideally driven by plain CSS (flexbox centering, `mask-image`) instead of
JS pixel math, with only one or two independent scroll measurements left in JS.

## What the section needs to do (agreed with the client over a long back-and-forth)

1. **Entry**: the section is one continuous scroll unit. When it scrolls into view,
   the whole thing pins ("sticky") vertically centered in the viewport — no separate
   sticky header/nav appears, just this section holding still.
2. **While pinned**: only two things visibly change as the user scrolls — the
   copy block (heading + description + button, cycling through 4 topics) and the
   product visual next to it. The section title ("Commerce begins before the cart."),
   subtitle, and the tab list on the left stay completely static (only the active
   tab's styling changes, tabs themselves never move or resize).
3. **The four topics**, in order: `ACCOUNT INTEGRITY`, `PURCHASE OPTIMIZATION`,
   `RETURNS ASSURANCE`, `LOREM IPSUM` (last one is a real placeholder in the
   existing content — reuse it, copy exists already in `.solutions-panel` markup).
4. **Copy behavior — continuous scroll, not a discrete swap**: the copy text should
   behave like a real scrolling column that's simply cropped to a fixed-height
   window (~700px tall as a starting point) with soft top/bottom edges (a
   `mask-image` gradient, not a hard `overflow: hidden` cut). As the user scrolls,
   copy for the current topic scrolls up and out through the top edge while the
   next topic's copy scrolls up and in from the bottom — one continuous motion tied
   1:1 to page scroll, not a fade/crossfade swap.
5. **Critical alignment rule**: text must be **fully opaque** exactly when it's
   aligned with the top of the product visual / the top of the tab list (call this
   "the alignment line"). The fade must happen **before** that line, in extra space
   built into the window above the alignment line — not fading right at/after the
   line. Concretely: the copy window's box should extend some buffer distance
   (e.g. ~96px) above the alignment line, and the mask gradient should reach full
   opacity exactly at that alignment line, with the fade-out only happening in that
   buffer region above it. (The client illustrated this with solid blocks + red
   gradients stacked over each column in Figma — the solid blocks were just their
   own visual guide for "this is the hard crop," not something to actually render;
   the red gradient is the real thing, i.e. the mask.)
6. **Image behavior — keep the existing technique, don't reinvent it**: the visual
   should keep the exact crossfade/push transition already implemented in the
   original `.solutions-wrapper` script (see "Reuse verbatim" below) — incoming
   image pushes in from the scroll direction while the outgoing one keeps moving
   and exits the same direction, like a carousel push. Do not change this mechanic,
   just port it to the new class names. Optionally add a light `mask-image` feather
   at the visual's own edges purely for polish (same fixed-height window pattern as
   copy), but the underlying push transition logic must stay the same.
7. **Exit**: once the user has scrolled through the last topic (`LOREM IPSUM`) and
   its copy has scrolled up to align with the same alignment line, the whole
   section should unstick and hand off to normal page scroll — no lingering pinned
   state, no dead extra scroll space before the next section (`.testimonials-block`)
   begins.
8. **Responsive**: below 900px, drop all of this — no sticky, no masks, no
   transforms. Just stack title → tabs (row, wrapped) → all 4 panels one after
   another with their own image each, exactly like the existing `.solutions-wrapper`
   and `.solutions-alt2-wrapper` responsive fallback already do (look at their
   `@media (max-width: 900px)` blocks for the pattern to mirror).

## Recommended architecture (far simpler than alt2 — please follow this shape
rather than re-deriving something closer to alt2's approach)

```
<section class="solutions4-wrapper section-divider">
  <div class="solutions4-scroller">              <!-- tall spacer; height = (panels + 1) * 100vh -->
    <div class="solutions4-stage">                <!-- position: sticky; top: 0; height: 100vh;
                                                         display:flex; flex-direction:column;
                                                         justify-content:center; -->
      <div class="solutions4-block grid-frame">
        <h2 class="solutions4-title">...</h2>
        <p class="solutions4-subtitle">...</p>
        <div class="solutions4-row">              <!-- 12-col grid, like the original -->
          <nav class="solutions4-tabs">...</nav>  <!-- span 2, plain static list -->

          <div class="solutions4-copy-window">     <!-- span 5, fixed-ish height + mask-image -->
            <div class="solutions4-copy-track">    <!-- absolutely positioned, transformed via JS -->
              <article class="solutions4-copy-panel" data-panel-index="0">...</article>
              <article class="solutions4-copy-panel" data-panel-index="1">...</article>
              <article class="solutions4-copy-panel" data-panel-index="2">...</article>
              <article class="solutions4-copy-panel" data-panel-index="3">...</article>
            </div>
          </div>

          <div class="solutions4-visual-window">   <!-- span 5 — same push-crossfade as original -->
            <div class="solutions4-visual-placeholder solutions4-visual-placeholder--active" data-panel-index="0">
              <img src="assets/assets/Img v2/solutions 01.jpg" alt="" class="solutions4-visual-image" />
            </div>
            <!-- ...index 1..3 identical to original's .solutions-visual-placeholder markup -->
          </div>
        </div>
      </div>
    </div>
  </div>
</section>
```

### Why this shape avoids alt2's fragility

- **Centering is pure CSS, no JS measurement**: `.solutions4-stage` is a single
  `position: sticky; top: 0; height: 100vh;` flexbox with `justify-content: center`.
  Whatever height the title/subtitle/tabs/copy/visual block actually renders at,
  flexbox centers it in the 100vh sticky box automatically — no measuring title
  height, no computed `padding-top`, no chain of `top` offsets to keep in sync.
- **One release point, not four**: because title, tabs, copy, and visual are all
  inside the *same single* sticky element (`.solutions4-stage`), they release
  together for free — there's no separate "rail height" bookkeeping per element
  like alt2's `leftRail` / `tabsRail` / `panelsFadeRail` / `visualRail`, each with
  its own cancel-out margin math. Just size `.solutions4-scroller` tall enough
  (`(panels + 1) * 100vh` is a good default — the last extra 100vh is pure runway
  for the native sticky release glide, so panel 4 gets its full dwell time fully
  pinned before the section releases) and the browser's native sticky behavior
  handles the rest.
- **Copy scroll is one ratio, not a dependency chain**: measure `copyTrack`'s
  total rendered height and `copyWindow`'s own height once (recompute on resize /
  `document.fonts.ready`, same milestones the original script already waits on).
  On scroll, compute a single `pageProgress` (0→1) from `.solutions4-scroller`'s
  own `getBoundingClientRect()` vs. its own height and the viewport height — one
  measurement, not several elements measuring each other — then set:
  `copyTrack.style.transform = translateY(-pageProgress * (trackHeight - windowHeight))`.
- **Active-topic detection reuses the original's own proven pattern, just
  re-rooted**: use the same `IntersectionObserver` "watch a line, whichever
  heading crosses it wins" technique the original section already uses for its
  scrollspy — but set `root: copyWindow` (not the viewport) and
  `rootMargin: '-50% 0px -50% 0px'` so it watches the center of the *small fixed
  window* instead of the whole page. `IntersectionObserver` correctly tracks
  elements through CSS `transform` on an ancestor, so this works even though
  `copyTrack` moves via `transform`, not native scrolling.
- **Tab click**: since panels have different text lengths, their scroll
  "budget" along the track isn't equal per panel — that's fine and actually
  correct (a longer panel should take a bit longer to scroll past). To jump to a
  topic on tab click: read that heading's own `offsetTop` within `copyTrack`
  (cache these once, same recompute milestones as above), solve backwards for the
  `pageProgress` that would put it centered in the window, then convert that back
  to an absolute page `scrollY` using `.solutions4-scroller`'s own top offset and
  height, and hand it to `lenis.scrollTo(...)` as the original script already does
  (with the same "re-assert active index once Lenis reports done" trick to dodge
  the race condition its comments describe).

### Reuse verbatim (don't rewrite this part)

The image push-crossfade in the original section's `<script>` — the
`pushPlaceholders` function, the `--active` / `--offscreen-top` class toggling,
the `solutions-visual-placeholder` CSS transitions (`transform: translateY(100%)`
base state, `translateY(-100%)` for `--offscreen-top`, `translateY(0)` for
`--active`, `transition: transform 0.6s cubic-bezier(0.65, 0, 0.35, 1)`) — copy
this whole mechanism over to `solutions4-visual-*` class names unchanged. It's
already correct and the client explicitly asked to keep it as-is.

### Mask-image values to start from

```css
.solutions4-copy-window {
  position: relative;
  overflow: hidden;
  margin-top: -96px;                 /* buffer runway sits above the alignment line */
  height: calc(64px + 96px + <window-height>); /* window-height ~ 60vh, tune visually */
  mask-image: linear-gradient(
    to bottom,
    transparent 0,
    black 96px,                      /* fully opaque exactly at the alignment line */
    black calc(100% - 56px),
    transparent 100%
  );
  -webkit-mask-image: /* same */;
}
```

Treat all pixel values above (96px buffer, 56px bottom fade, ~60vh window height)
as starting points to tune by eye once this is live in a real browser — they
were derived from reasoning about the Figma redlines, not from a pixel-measured
spec, and the client cares a lot about the "fully opaque at the alignment line"
feel reading correctly, so plan to iterate on those numbers visually.

## Suggested build order
1. Copy `.solutions-wrapper`'s HTML block, rename every class to `solutions4-`,
   restructure per the architecture above (stage wrapper, copy-window/copy-track
   split, single scroller).
2. Port the CSS: static title/subtitle/tabs styles copy over almost unchanged;
   write new rules for `.solutions4-scroller`, `.solutions4-stage`,
   `.solutions4-copy-window/-track/-panel`; port `.solutions-visual-*` rules
   verbatim to `.solutions4-visual-*`.
3. Write the JS: single scroll listener computing `pageProgress` → drives
   `copyTrack` transform; `IntersectionObserver` rooted in `copyWindow` → drives
   `setActive(index)` (tab class + image crossfade, reusing `pushPlaceholders`);
   tab click handler computing target scroll via the formula above.
4. Add the `@media (max-width: 900px)` fallback (stack everything, kill sticky/
   transform/mask), mirroring the existing responsive blocks.
5. Set `.solutions-alt2-wrapper`'s opening tag to
   `style="display:none" data-hidden-for-debug="true"` so `solutions4` is the one
   that actually shows.
6. **Test in a real browser, scrolling by hand**, at a few viewport heights
   (short laptop, tall desktop) — this is exactly the step that couldn't be done
   in the sandboxed environment this brief came from. Confirm: centering looks
   right at different viewport heights, text reads fully opaque right at the
   image/tabs alignment line, the image push transition still looks identical to
   the original, and the section releases cleanly with no dead scroll gap before
   testimonials.
