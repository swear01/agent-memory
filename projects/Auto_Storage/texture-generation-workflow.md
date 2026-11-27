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

- **Active 的 RGB 總量較高，F11 看起來還是分不出狀態。** 先選已接受的 Auto Storage chassis／palette；不要分開生成兩張全 tile。以固定 `core_mask` 外保留原 chassis bytes，再在較小 `core_visual_mask` 裡從同一 inactive 成品派生 active role 色。Fusion connected sheet 必須從最終兩種 native states 各自生成。
- **只以亮度大於另一張就判 PASS。** 將 normalized sRGB channel 在 ≤0.04045 時除 12.92，其餘 `((c+0.055)/1.055)^2.4`，以 `Y=0.2126R+0.7152G+0.0722B` 計 visual-core mean。Flux Station gate 為 inactive ≤0.10、active ≥0.35、比值 ≥3.0，且 core 外每個 pixel 都與 chassis 和另一狀態一致。實機若仍難分，數值通過也要退回。
- **只有 native PNG 通過，連接圖或實機仍用舊素材。** 保存 model/seed/prompt/reference/strength、source hashes、masks、role map、state 派生關係、亮度門檻與 native/connected hashes 到 `state_manifest.json`。先跑 focused texture tests，再全套 Python、`./gradlew runData`、`./gradlew build`、Flux Networks GameTest 與最新成品的 F11 目視。

[原始紀錄](https://github.com/swear01/agent-memory/blob/9b2d556e370f3a71736d5dc0a63f3b1e1c8c3a0e/projects/Auto_Storage/texture-generation-workflow.md)
