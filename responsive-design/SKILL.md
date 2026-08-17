---
name: responsive-design
description: Build fluid, mobile-first layouts that adapt gracefully from small phones to large desktops using flexible units, CSS Grid/Flexbox, container queries, and content-driven breakpoints; use when a UI breaks on certain screen sizes, when starting a new layout, when making a design responsive, or when users mention mobile, breakpoints, viewport, fluid typography, or media queries.
---

# Responsive Design

This skill describes a systematic, mobile-first approach to building layouts that work across the full range of devices and viewport sizes. It emphasizes content-driven breakpoints, flexible units, and modern CSS layout primitives over pixel-perfect fixed designs.

## When to use this skill

- Starting a new page or component that must work on phones, tablets, and desktops.
- An existing layout overflows, overlaps, or looks broken at certain widths.
- Converting a fixed-width design to be responsive.
- Users mention breakpoints, viewport, mobile-first, fluid type, container queries, or media queries.

## Instructions

1. **Start mobile-first.** Write base styles for the smallest viewport, then layer enhancements at larger widths with `min-width` media queries. This keeps CSS additive and avoids overrides.
2. **Set the viewport meta tag.** Ensure `<meta name="viewport" content="width=device-width, initial-scale=1">` is present so mobile browsers render at device width.
3. **Prefer flexible units.** Use relative units (`rem`, `em`, `%`, `fr`, `ch`, `vw`, `vh`) over fixed `px` for sizing, spacing, and typography. Reserve `px` for hairline borders and small fixed details.
4. **Choose breakpoints from content, not devices.** Add a breakpoint when the layout starts to break, not at arbitrary device widths. Keep the set small and named by intent.
5. **Use modern layout primitives.** Reach for Flexbox for one-dimensional flow and CSS Grid for two-dimensional layouts. Use `minmax()`, `auto-fit`/`auto-fill`, and `clamp()` to build layouts that flex without many media queries.
6. **Adopt container queries for components.** When a component appears in multiple contexts, size it based on its container (`@container`) rather than the viewport, so it is truly reusable.
7. **Make media fluid.** Set `img, video { max-width: 100%; height: auto; }`, use `srcset`/`sizes` for responsive images, and provide `aspect-ratio` to prevent layout shift.
8. **Scale typography fluidly.** Use `clamp(min, preferred-vw, max)` for headings so text scales smoothly between breakpoints without abrupt jumps.
9. **Respect touch and input.** Ensure tap targets are at least ~44x44px, avoid hover-only interactions, and test with both touch and pointer.
10. **Test the range.** Verify at ~320px, ~768px, ~1024px, and ~1440px plus in-between widths. Check landscape, zoom to 200%, and honor `prefers-reduced-motion`.

## Examples

Responsive card grid with no media queries:

```css
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(16rem, 1fr));
  gap: clamp(1rem, 2vw, 2rem);
}
```

Fluid heading that scales between a floor and ceiling:

```css
.title {
  font-size: clamp(1.5rem, 1rem + 3vw, 3rem);
  line-height: 1.1;
}
```

Container query so a component adapts to its parent, not the viewport:

```css
.media-object {
  container-type: inline-size;
}

@container (min-width: 30rem) {
  .media-object__inner {
    display: grid;
    grid-template-columns: auto 1fr;
    gap: 1rem;
  }
}
```

## Checklist

- [ ] Viewport meta tag present.
- [ ] Base styles are mobile-first; enhancements use `min-width`.
- [ ] Layout uses relative units and Grid/Flexbox, not fixed widths.
- [ ] Breakpoints chosen where content breaks, kept minimal.
- [ ] Images/video are fluid with `max-width: 100%` and `aspect-ratio`.
- [ ] Typography scales with `clamp()` where appropriate.
- [ ] Tap targets >= 44px; no hover-only critical actions.
- [ ] Verified at 320/768/1024/1440px, landscape, and 200% zoom.
