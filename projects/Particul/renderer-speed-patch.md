---
title: Particul Electron renderer speed patch
scope: project
status: active
updated: 2026-09-04
project: Particul
tags: [electron, chromium, webgpu, openspeedy, speedhack]
---

- **OpenSpeedy on Electron main leaves the game unchanged or one subsystem still slow.** Visible particles advance in Chromium's renderer via frame-step WGSL, JavaScript tick drives autosellers/extractors/the month, and Plinko has its own compute and clock. Measure the actual renderer and each loop; built-in speed-up only changes `monthLength`. Preserve the original bundle and validate the replacement via `node --check` and real Steam relaunch.
- **Extractor attempts many particles but output stays low.** Multiple writes to the same emitter cell before compute overwrite each other. Queue and bound same-cell spawns, interleaving each accepted write with a movement iteration. A command buffer per iteration caused renderer memory growth and black canvas despite a responsive process. Validate both actual canvas and working set.
- **Particle counters show zero or autosell seems unaccelerated.** Submit the count/render command buffer before calling `readBack()`. The count feeds UI as well as `removeLastParticles`, so verify labels and resource movement separately.
- **Save stalls after acceleration or cloud import blocks startup.** The simulated month can schedule unawaited GPU full-buffer readback and IndexedDB writes faster than they finish. Throttle autosave by real time (verified 5 s), verify local `saves/main` plus particle blob, keep original cloud files, and load verified local state before waiting for cloud import.
- **Marketplace patch passes static checks but slot/wheel still animate.** Locate the active component from the view; the first patch targeted duplicate bundles. Preserve original weighted result, costs, callbacks and achievement side effects when removing presentation delays. Steam cache was verified 27/27 after a diagnostic achievement trigger. Preserve that provenance when evaluating normal play.

[完整原文](https://github.com/swear01/agent-memory/blob/06f50e3a42e850b37cffb6b6a4a446ef670e9625/projects/Particul/renderer-speed-patch.md)
