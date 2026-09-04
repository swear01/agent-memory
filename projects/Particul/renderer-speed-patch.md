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

Extractor generation has a separate bottleneck: the scaled `tick(e)` can call
`addParticleAt` many times in one JavaScript frame, but every call writes one
particle to the same extractor cell with `queue.writeBuffer`. The writes do not
accumulate into multiple cells before the following GPU compute pass; later
writes overwrite the same cell. Increasing the timer scale therefore increases
the attempted spawn count without increasing the visible particle count. A
real extractor-rate fix needs a spawn queue/buffer or interleaved GPU simulation,
not only a larger timer multiplier.

The applied extractor fix adds `pendingSpawns` and queues extractor writes.
Each of the 50 main simulation iterations flushes at most one queued spawn per
unique emitter cell, submits that write, and then submits its compute pass.
This interleaves movement with spawning so repeated writes to one emitter cell
no longer overwrite one another. The patched bundle passed `node --check` and
the Renderer remained responsive after relaunch.

Submitting one command buffer per compute iteration caused the Electron
Renderer working set to grow rapidly and produced a black canvas while the
Renderer still reported as responsive. The stable repair keeps one command
buffer per frame, retains the 50-iteration shader loop for the small Plinko
simulation, and bounds duplicate extractor queue entries. On relaunch the
Renderer stayed responsive and its working set stopped growing during the
runtime check.

After interleaving 50 compute submissions, the count pass is submitted in a
later command buffer. `readBack()` must therefore be called after that count
buffer is submitted; calling it before submission reads cleared/stale counts.
Those counts drive both the top-left particle labels and `removeLastParticles`
used by auto-sellers, so an early readback makes the labels empty/zero and
makes auto-selling appear to return to normal speed. Moving `readBack()` after
the count/render submission fixes both symptoms.

The 50x timer also scales the monthly autosave trigger. The renderer's
`save()` copies the full WebGPU particle buffer to a MAP_READ buffer and sends
it to a worker-backed IndexedDB writer; the worker emits a shared `saved`
event. Calling this unawaited save once per simulated month floods GPU readback
and IndexedDB transactions, which can stall the game and leave startup unable
to finish loading. Throttling autosave to once per 5 real seconds prevents the
flood. The game also stalled during startup while awaiting
`tryImportSaveFromCloud()` even though the Steam Cloud files were intact;
bypassing that blocking import and loading local IndexedDB first restored the
renderer loop. Before repair, the local IndexedDB contained the `saves/main`
record and a particle blob; Steam Cloud retained `saveStrings.sav` and
`saveBuffer.sav` with the original progress.
