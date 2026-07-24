# n-body

A real-time N-body gravity simulation in the browser, using a **WebGPU** compute
shader to do genuine O(n&sup2;) pairwise Newtonian gravity - every body pulls on
every other body, every frame - rather than an analytical shortcut or a faked
visual approximation.

Open `index.html` directly in a WebGPU-capable browser (Chrome or Edge 113+ on
desktop). No build step, no server, no network access needed - Three.js is
bundled locally as `three.min.js` and used only as a math/camera-orbit helper;
all the actual simulation and rendering is raw WebGPU (WGSL compute + render
pipelines).

## What's actually happening

Three massive central bodies start on a ring, each with its own mini-disk of
lighter bodies on near-circular orbits, and the whole system is left to evolve
under real mutual gravity - nothing here is scripted. It's deliberately *not*
a stable configuration (symmetric N-body rings with 3+ bodies generally
aren't, even tuned carefully - a "Klemperer rosette" is a famous example of a
balanced-but-unstable case), so expect real drift: mini-disks warping, density
waves, and material getting perturbed between the central bodies over time.

Physics per frame, in `index.html`'s WGSL compute shader:

1. For every body, sum the gravitational acceleration from every other body
   (softened at very close range to avoid a singularity).
2. Integrate with semi-implicit (symplectic) Euler - velocity updates from the
   current acceleration, then position updates from the *new* velocity, which
   is noticeably more stable for orbital motion than naive explicit Euler at
   the same step size.

Rendering is a second WebGPU pipeline: each body becomes a camera-facing
billboard quad (instanced, sized by mass) with a soft circular falloff,
colored by speed (blue-ish for slow, warm/white for fast).

## Quality presets

Cost scales with the *square* of body count (O(n&sup2;)), so particle count is
the one setting that matters most for whether a given GPU keeps it smooth -
same idea as graphics-quality presets in a game. Tested on integrated graphics
(Intel Core Ultra 5 125U, Arc iGPU, no dedicated GPU):

| Preset  | Bodies | Measured FPS |
|---------|-------:|-------------:|
| Low     |  1,500 | 60 (stable)  |
| Medium  |  4,000 | -            |
| High    |  8,000 | -            |
| Ultra   | 16,000 | 49           |
| Extreme | 28,000 | ~15 (still renders correctly, just slower) |

Drag to orbit the camera, scroll to zoom.

## Why WebGPU specifically

WebGL (used by the sibling project this was split out of,
[particles](https://github.com/robertclemo/screensavers)) has no
general-purpose compute shaders, so there'd be no way to do real N-body
physics at more than a token particle count without falling back to an
analytical shortcut. WebGPU's compute pipelines are what make genuine O(n&sup2;)
pairwise gravity feasible in real time in a browser tab.
