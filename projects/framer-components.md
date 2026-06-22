# Framer Code Components

Interactive UI components built with React + TypeScript, deployed as Framer code components on petermarc.com.
These are production components — not experiments. Each solves a specific interaction problem with real device APIs.

---

## Holo Carousel (`cardsHolo`)

Five-card holographic carousel with Pokémon-style shimmer and full mobile sensor support.

### What it does

```
Cards:    5 unique cards in a curved arc layout
Shimmer:  Rainbow + shine + glare blend-mode layers masked to card silhouette
Tilt:     DeviceOrientation API drives shimmer on mobile (iOS permission-gated)
Haptics:  iOS switch trick (input[type=switch] label click) + Android Vibration API
Pointer:  Mouse/stylus drives 3D tilt + holo effect on active card
Swipe:    Touch/pen swipe gestures with velocity detection
Keyboard: ← → arrow key navigation
```

### Architecture

Ghost-duplicate infinite scroll: 3 copies × 5 cards = 15 nodes in DOM. When the active
index drifts into a ghost copy, a silent zero-transition re-center snaps it back to the
middle copy — visually invisible because adjacent cards are identical across copies.

Arc layout uses trigonometry: each card offset from center gets an angle × radius → x/y/z
transform. Cards fade to opacity 0 beyond ±2 offset.

Blend mode stack (all masked to card image):
1. Rainbow layer — repeating gradient + exclusion blend → `mix-blend-mode: overlay`
2. Shine layer — radial gradient at pointer position → `mix-blend-mode: soft-light`
3. Glare layer — tighter radial gradient → `mix-blend-mode: overlay`

CSS custom properties drive everything: `--pointer-x/y`, `--background-x/y`, `--rotate-x/y`,
`--card-opacity`, `--arc-x/y`, `--arc-rotate`, `--arc-scale`, `--device-tilt-x/y`.

### Notable decisions

- `isolation: isolate` on the inner container — prevents Safari from recompositing the
  entire parent stack on every opacity tween (eliminates a persistent flicker bug)
- `filter: drop-shadow` on the image element, not the wrapper — shadow on the parent
  triggers per-frame shadow recalculation on blend mode changes in Safari
- z-index intentionally excluded from CSS transitions — active card must pop above
  outgoing card instantly to prevent mid-transition flash

### Property controls (Framer panel)

```
Card 1–5 images · Start Index · Background · Card Height · Arc Spread
Float Strength · Arrows on/off · Indicators on/off · Arrow Color
Indicator Color · Haptics on/off · Tilt Holo on/off · Diagnostics on/off
```

---

## Icon Burst (`iconBurst`)

Tap the main app icon — secondary icons burst outward with physics and fly off screen.

### What it does

```
Tap:      Main icon pops (scale bounce) → spawns N random secondary icons
Physics:  Each chip gets random angle, speed, gravity (0.32 px/frame), rotation velocity
Cleanup:  Chips self-remove when they leave the viewport — no memory leak
Hover:    Main icon scales to 1.05 on hover
Props:    Up to 11 burst icons, min/max burst count, icon size, border radius, hint text
```

### Architecture

Chips are created as raw DOM nodes (not React state) and appended to a fixed-position
overlay layer. Physics runs in a `requestAnimationFrame` loop; when the chip exits the
viewport bounding box it calls `el.remove()`. This keeps React render cycle out of the
hot path — 60fps even with 11 simultaneous chips.

Pop animation on the main icon uses a CSS keyframe via `animation:` property swap
(not a transition) so it can re-trigger on each tap without needing to reset state first.

Border radius auto-matches iOS icon spec: `~27%` of icon size for main, `~23%` for chips.
Both are overridable in the Framer panel.

### Property controls (Framer panel)

```
Main Icon · Burst Icons (array, max 11) · Icon Size · Main Radius
Chip Radius · Min Burst · Max Burst · Show Hint · Hint Text · Background
```

---

[← Back to profile](../README.md)
