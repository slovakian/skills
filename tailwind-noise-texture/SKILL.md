---
name: tailwind-noise-texture
description: Adds SVG feTurbulence noise grain overlays with Tailwind CSS. Use when adding noise texture, film grain, or grainy backgrounds to UI.
---

- Parent: `relative overflow-hidden`
- Add as last child:

```html
<figure aria-hidden="true" class="absolute inset-0 z-10 pointer-events-none filter-[url('#noise-bg-fx')_grayscale(100%)] opacity-10 mix-blend-screen">
  <svg class="size-0" aria-hidden="true">
    <filter id="noise-bg-fx">
      <feTurbulence baseFrequency="0.8" />
    </filter>
  </svg>
</figure>
```

- Unique filter `id` per overlay on the page
- Dark backgrounds: `mix-blend-screen opacity-10`
- Light backgrounds: `mix-blend-multiply opacity-[0.08]`
- Finer grain: `baseFrequency="1.2"`. Coarser: `baseFrequency="0.5"`
