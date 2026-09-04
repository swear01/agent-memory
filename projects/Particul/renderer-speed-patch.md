---
title: Particul Electron renderer speed patch
scope: project
status: active
updated: 2026-09-04
project: Particul
tags: [electron, chromium, webgpu, openspeedy, speedhack]
---

# Particul Electron renderer speed patch

Particul is an Electron game whose visible simulation runs in a Chromium
Renderer process, separate from the main process. OpenSpeedy injected into the
main process did not inject into the Renderer, so changing the main process
speed did not affect the simulation.

The main simulation advances particles with one WebGPU compute dispatch per
`requestAnimationFrame`; its WGSL shader is frame-step based and has no
`deltaTime`. The JavaScript `tick(e)` separately advances auto-clickers,
auto-sellers, mouse drill, and the month clock. The built-in `speed up` action
only changes `monthLength` from 5 to 1.

The Plinko mini-game has an independent WebGPU compute dispatch and animation
clock. A game-specific speed patch therefore needs both the main simulation and
Plinko compute loops changed, plus the JavaScript timer scale. The verified
50x patch was applied to the bundled renderer script, with an adjacent backup
of the original bundle.
