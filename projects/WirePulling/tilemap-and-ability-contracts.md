---
title: WirePulling tilemap and movement contracts
scope: projects/WirePulling
project: WirePulling
tool: Unity
status: active
confidence: high
evidence: >-
  Current project source, Unity EditMode coverage, and focused PlayMode coverage
  define the four-layer Tilemap, one-way collision, hazard/thin-wall, and
  ThunderDrop contracts.
created: 2026-08-20
updated: 2026-08-20
tags:
  - unity
  - tilemap
  - physics
  - platformer
  - thunder-drop
source_refs:
  - WirePulling:Assets/Tests/EditMode/TilemapLevelBuilderTests.cs
  - WirePulling:Assets/Tests/PlayMode/OneWayPlatformRuntimeTests.cs
  - WirePulling:Assets/Tests/PlayMode/ParkourThunderDropInputTests.cs
  - WirePulling:Assets/Tests/EditMode/GameplayBugRegressionTests.cs
  - WirePulling:docs/Unity_Development_Workflow.md
redaction: passed
generated_by: openai-codex/gpt-5.6-luna
---

# Tilemap source of truth

Treat the scene Tilemap as authoritative. ASCII/text export is a read-only AI
mirror of the active Tilemap; do not use the old level-data cache as the write
source. The four gameplay layers have separate contracts:

- Terrain owns solid walls, full-cell platforms, and terrain composite geometry.
- OneWay owns thin visual platforms, a `PlatformEffector2D`, and composite
  collision configured for one-way behavior.
- Logic owns hazard tiles with trigger collision and `HazardPlatform` behavior.
- ThinWall owns `ThinRedWall` tiles with direct solid collision, the dedicated
  layer/tag, and no trigger behavior.

After editing a Tilemap, process tilemap changes, generate collider geometry,
and synchronize physics before saving. A used-tile count alone is insufficient;
the populated physical layer must also have generated composite paths.

# Runtime movement

A one-way platform must allow a dynamic body moving upward from below to pass
through. A grounded player pressing down temporarily ignores the standing
one-way collider, is pushed downward, and has that collision restored after the
drop window.

ThunderDrop is not a default Parkour unlock. The level grants it through its
ability provider. With the ability granted, downward input plus the configured
action starts ThunderDrop while airborne, takes priority over a normal Dash,
and applies a fast downward velocity. Teardown must restore the temporary
ThinRedWall collision override.

The focused tests cover layer placement and collider modes, one-way pass/drop
behavior, default-lock behavior, Dash-priority behavior, and teardown cleanup.
