---
title: Auto Storage state texture generation requires a perceptual luminance gate
scope: projects/Auto_Storage
project: Auto Storage
tool: Minecraft/NeoForge, Replicate rd-fast, QMD
status: active
confidence: high
evidence: >-
  Auto Storage Flux Station #154 source assets, deterministic assembler,
  focused texture/static tests, full Python suite, runData, build, and the
  isolated Flux Networks GameTest were verified after the correction.
created: 2026-08-22
updated: 2026-08-22
tags:
  - auto-storage
  - minecraft
  - texture-generation
  - pixel-art
  - luminance
  - qmd
source_refs:
  - Auto_Storage:docs/texture-generation-workflow.md
  - Auto_Storage:docs/superpowers/specs/2026-08-22-flux-station-state-texture-design.md
  - Auto_Storage:art/texture-generation/20260822-flux-station/assemble_flux_station.py
  - Auto_Storage:scripts/test_flux_station_textures.py
  - Auto_Storage:scripts/test_static_regressions.py
redaction: passed
---

# Verified lesson

The first Flux Station state pair used a raw RGB channel-sum check and a
`1.2x` active brightness mapping. Automation reported `active > inactive`, but
the first F11 review found the state difference too weak to identify. The
measured raw-sum difference was only about `1.24x`. A greater-than assertion is
not a sufficient visual-state contract.

## Required generation boundary

Generate candidates with the approved `retro-diffusion/rd-fast` workflow, but
never generate active and inactive as independent full-tile predictions. Select
the existing Auto Storage family chassis and palette as the source of truth.
Declare a fixed central `core_mask`; copy the chassis byte-for-byte outside it.
Declare a smaller `core_visual_mask` for the actual emissive pixels. Compose the
inactive state with a deterministic dark role palette, then copy that completed
image to derive active and replace only the same visual-mask pixels with bright
role colors. Generate Fusion connected sheets only from those final native
state images.

## Objective brightness gate

Use sRGB relative luminance, not raw channel sums:

```text
channel = sRGB / 12.92                         if sRGB <= 0.04045
        = ((sRGB + 0.055) / 1.055) ^ 2.4      otherwise
Y = 0.2126R + 0.7152G + 0.0722B
```

The current Flux Station gate is:

- inactive visual-core mean `Y <= 0.10`;
- active visual-core mean `Y >= 0.35`;
- active/inactive ratio `>= 3.0`;
- every pixel outside `core_mask` is identical between states and identical to
  the selected chassis.

The corrected verified measurements are inactive `0.021387`, active `0.377104`,
and ratio `17.632298`. The numeric gate is necessary but does not replace the
human F11 verdict: if the state still looks ambiguous, the asset fails.

## Provenance and verification

Persist model, seed, prompt, reference, strength, source hashes, both masks,
role color mapping, luminance method/thresholds/measurements, parent-state
relationship, output paths, and native/connected hashes in the generation
metadata and `state_manifest.json`. Run the focused texture test first, then
the full Python suite, `./gradlew runData`, `./gradlew build`, the relevant
GameTest fixture, and a fresh exact-artifact F11 review.

The correction was verified with 201 focused texture/static tests, 642 total
Python tests, datagen with zero written drift, a successful Gradle build, and
Flux Networks GameTest `20/20`.
