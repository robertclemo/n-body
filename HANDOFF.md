# n-body

## What this is
A real-time N-body gravity simulation that runs entirely in the browser using a **WebGPU compute shader** to do genuine O(n²) pairwise Newtonian gravity — every body pulls on every other body, every frame, not an analytical shortcut or a faked visual effect. Three massive central bodies on a ring, each with its own mini-disk of lighter orbiting bodies, left to evolve under real mutual gravity (deliberately not a stable configuration, so visible drift/density-waves/perturbation over time is expected behavior, not a bug).

This project was split out of the `particles` project (see that project's README) specifically because it needed WebGPU compute shaders, which WebGL (used everywhere else in `particles`) doesn't support.

## Status
Working, per the project's own README, which documents measured FPS across quality presets on a specific test machine (Intel Core Ultra 5 125U / Arc integrated graphics): Low (1,500 bodies) holds a stable 60fps; Ultra (16,000 bodies) holds 49fps; Extreme (28,000) drops to ~15fps but still renders correctly. Two commits total — an initial commit and the split-out from `particles`.

## Getting it running (concrete commands)
No build step, no server needed:
```
Open index.html directly in a WebGPU-capable browser (Chrome or Edge 113+, desktop).
```
`three.min.js` is bundled locally (not a CDN reference) and is only used as a math/camera-orbit helper — the actual simulation and rendering are raw WebGPU (WGSL compute + render pipelines), confirmed by reading `index.html`. There's a `#fallback` div in the HTML for browsers without WebGPU support.

## Structure
- `index.html` — everything: WGSL compute shader for physics, WGSL render pipeline for the billboard-quad rendering, HUD (particle count, FPS, quality presets), camera orbit/zoom controls.
- `three.min.js` — bundled Three.js, used only for camera math, not rendering.
- `README.md` — thorough, written by a prior session; already covers physics approach, quality presets/FPS, and the "why WebGPU" rationale in more depth than needed here.

## Design decisions worth knowing
- Integration uses semi-implicit (symplectic) Euler — velocity updates from current acceleration, then position updates from the *new* velocity — explicitly chosen over naive explicit Euler for better orbital stability at the same step size.
- Close-range gravity is softened to avoid a singularity when two bodies get very close.
- Rendering: each body is a camera-facing billboard quad, instanced, sized by mass, colored by speed (blue-ish slow, warm/white fast).
- Cost scales with body count squared, so the quality presets exist specifically because particle count is the dominant performance lever on weaker GPUs.

## Known issues / open risks
- Requires WebGPU support (Chrome/Edge 113+ desktop) — will not run at all in browsers without it, beyond showing the fallback message.
- No automated tests; correctness is validated visually/empirically (density waves and structure "emerging" is treated as evidence physics is working correctly).

## Next steps
None documented beyond what already shipped. The README frames this as a complete, working split-out of a scene that outgrew its parent project.

## Useful context
- Git remote: `https://github.com/robertclemo/n-body.git`
- Full history (`git log --oneline -10`):
  ```
  5450a52 Split out of the particles repo: real WebGPU N-body gravity sim
  9437b81 Initial commit
  ```
- Sibling/parent project: `particles` (`C:\Users\rober\Documents\particles`, remote `robertclemo/screensavers`) — that project's README explicitly documents this split and the reasoning behind it.
