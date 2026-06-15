---
name: cross-platform-animations
description: Core principles and implementation strategies for building highly interactive, multi-stage, cross-platform animations using CSS/React and Compose Multiplatform (Kotlin). Use when tasked to create complex layered UIs, visual tear/split effects, or rich sweeping highlights.
---

# Advanced Cross-Platform UI Animations Specialist

This skill outlines the necessary abstractions and patterns for building premium, multi-stage interactive UI animations across both Next.js/React (web) and Compose Multiplatform (native).

## Core Concepts

When building rich visual interactions (e.g. unboxing, revealing hidden objects, splitting elements, 3D card flips), the architecture relies on synchronized state machines, precise Z-index layering, and shape clipping.

### 1. State-Driven Stages
Complex interactions rarely transition between two states. Use numerical states or strict enums to control the progression.
- `Stage 0 (Idle)`: Initial appearance.
- `Stage 1 (Transition/Preparation)`: An intermediate phase (e.g., a "cutting" laser, a wind-up effect).
- `Stage 2 (Action)`: The core physical translation or rotation.
- `Stage 3 (Resolution)`: The final showcase or continuous looping idle animation.

### 2. Layering (Z-Index and Depth)
Instead of relying on absolute flat positioning, stack elements in a pseudo-3D space.
- Background/Environment: `z-index: 10`
- The Revealed Object (Middle Layer): `z-index: 20`. This layer usually starts hidden *behind* the foreground, transitioning upward/outward to become the focal point.
- The Foreground/Shell: `z-index: 30`. This layer visually covers the object, and is usually removed, dropped, or translated off-screen during the reveal.

### 3. Splitting & Clipping Paths
Do not use separate distinct image assets for a unified object that splits. Use a single unified visual asset and apply clipping paths to generate top and bottom components that fit together flawlessly.
- **Web (CSS)**: `clip-path: polygon(...)`
- **Compose**: `.clip(GenericShape { size, _ -> ... })`
By carefully matching the coordinates (e.g. splitting an element diagonally across `15%` to `18%`), you can translate the separate halves apart independently.

### 4. Continuous Ambient Highlights (Holographic/Glow)
Once a premium asset is revealed, static presentation is insufficient. Use color blending to simulate dynamic light sources (e.g. holographic sweeps or ambient glows).
- **Web**: Apply a `mix-blend-color-dodge` or `mix-blend-overlay` layer absolutely positioned over the object. Animate the `background-position` of a linear gradient.
- **Compose**: Apply a `Brush.linearGradient` to a Box overlaying the component, and animate the `start` and `end` offsets using `rememberInfiniteTransition()`.

## Implementation Examples

### Multi-Platform Shape Clipping
**Compose:**
```kotlin
val topPieceShape = GenericShape { size, _ ->
    moveTo(0f, 0f)
    lineTo(size.width, 0f)
    lineTo(size.width, size.height * 0.45f) // The cut angle
    lineTo(0f, size.height * 0.40f)
    close()
}
// Apply with Modifier.clip(topPieceShape)
```

**Next.js (Tailwind):**
```tsx
<div style={{ clipPath: 'polygon(0 0, 100% 0, 100% 45%, 0 40%)' }}>...</div>
```

### 3D Flipping and Depth
**Next.js:**
Utilize `[perspective:2000px]` on the container, `[transform-style:preserve-3d]` on the wrapper, and `[backface-visibility:hidden]` on the front and back faces. Flip the inner element via `rotateY(180deg)`.
**Compose:**
Utilize `.graphicsLayer { rotationY = 180f; cameraDistance = 16f * density }`.

### Progressive Transformations
Link interactions to states using staggered transitions:
**Compose:**
```kotlin
val slideOffset by animateFloatAsState(
    targetValue = if (stage >= 2) -50f else 0f,
    animationSpec = tween(durationMillis = 1500, easing = FastOutSlowInEasing)
)
```
**Next.js (Tailwind Arbitrary Durations):**
```tsx
className={`transition-all duration-[1500ms] ease-[cubic-bezier(0.23,1,0.32,1)] ${
  stage >= 2 ? "-translate-y-10 scale-110" : ""
}`}
```
