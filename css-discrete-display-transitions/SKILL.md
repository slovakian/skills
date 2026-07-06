---
name: css-discrete-display-transitions
description: Fade panels in/out when toggling display:none with @starting-style and transition-behavior allow-discrete. Use for responsive sidebars, drawers, and popovers without JS. Covers pure CSS and Tailwind v4 (starting:, transition-discrete).
---

## Problem

`display: none` ↔ visible snaps instantly. Opacity-only hacks leave elements in the accessibility tree. JS resize listeners add complexity.

## Solution

Two CSS primitives work together:

1. **`@starting-style`** — defines the “from” state when an element first becomes visible (including leaving `display: none`).
2. **`transition-behavior: allow-discrete`** (via `allow-discrete` on the transition shorthand) — defers flipping discrete properties (`display`, `visibility`) until continuous properties (e.g. `opacity`) finish animating.

## Pure CSS — responsive panel (resize fade)

```css
.panel {
  display: none;
  opacity: 0;
  transition: opacity 0.2s, display 0.2s allow-discrete;
}

@media (min-width: 1280px) {
  .panel {
    display: flex;
    opacity: 1;
  }

  @starting-style {
    .panel {
      opacity: 0;
    }
  }
}
```

- Base: hidden + transparent.
- At breakpoint: show + opaque; `@starting-style` fades **in** on entry.
- Below breakpoint: reverses; `allow-discrete` on `display` fades **out** instead of popping.

## Tailwind CSS v4

Tailwind v4 supports both primitives:

| CSS | Tailwind v4 |
| --- | --- |
| `transition-behavior: allow-discrete` | `transition-discrete` |
| `@starting-style { … }` | `starting:` variant |

Docs: [transition-behavior](https://tailwindcss.com/docs/transition-behavior), [starting variant](https://tailwindcss.com/docs/hover-focus-and-other-states#starting-style).

### Responsive panel (Tailwind)

```html
<aside
  class="hidden opacity-0 transition-opacity duration-200 transition-discrete xl:flex xl:opacity-100 xl:starting:opacity-0"
>
  <!-- panel content -->
</aside>
```

- `hidden` / `xl:flex` — discrete show/hide at breakpoint.
- `transition-discrete` — required for exit fade on `display`.
- `xl:starting:opacity-0` — entry fade scoped to the breakpoint (mirrors `@starting-style` inside the media query).

### Popover / dialog (Tailwind)

```html
<div
  popover
  id="menu"
  class="opacity-0 transition-all duration-300 transition-discrete open:opacity-100 starting:open:opacity-0"
>
  <!-- content -->
</div>
```

Pair `starting:` with a state variant (`open:`, `peer-checked:`, etc.) when visibility is toggled by attribute/state, not media query.

## JSX / React

**Yes — use this in JSX.** The animation is CSS-driven; React does not need resize listeners or transition libraries for responsive panels.

```tsx
<aside className="hidden opacity-0 transition-opacity duration-200 transition-discrete xl:flex xl:opacity-100 xl:starting:opacity-0">
  {children}
</aside>
```

Guidelines:

- **Prefer Tailwind utilities** in `className` — no extra CSS file needed.
- **CSS Modules / global CSS** — put `@starting-style` in a `.css` file; reference the class in JSX. `@starting-style` cannot be expressed with inline `style={{}}`.
- **Avoid** toggling `opacity-0 pointer-events-none` instead of `hidden` just to animate — screen readers still traverse non-`display:none` elements.
- **Do not** reach for Framer Motion / JS for simple responsive show-hide fades unless you need orchestrated multi-element choreography or unsupported browsers.

Styled-components / Emotion: `@starting-style` works inside tagged-template CSS blocks.

## Browser support

- Chrome / Edge 117+
- Firefox 129+
- Safari 17.5+

Below these versions, panels still show/hide correctly — they just pop without fade. No JS fallback required unless polish on old browsers is mandatory.

## When to use / skip

| Use | Skip |
| --- | --- |
| Responsive sidebars, filter panels, nav drawers | Complex staggered list animations |
| Popovers, dialogs, disclosure toggles | Need to animate `height: auto` (use `@keyframes` or JS) |
| Zero-JS fade on `display` toggle | Tailwind v3 projects (no `starting:` / `transition-discrete`) |

## Checklist

- [ ] Base state sets both `display: none` (or `hidden`) **and** `opacity: 0`
- [ ] Transition list includes `display` with `allow-discrete` (or `transition-discrete` in Tailwind)
- [ ] `@starting-style` / `starting:` defines entry opacity (scoped to breakpoint or state variant)
- [ ] Exit fade verified by resizing viewport or toggling state — not just entry
