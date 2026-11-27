---
title: CBDK 8M 供電環 DRC 的本機重播與雙切孔
scope: projects/cvsd-final-sce
status: verified
updated: 2026-11-28
tags: [cbdk, icv, drc, pg, valkyrie]
---

# CBDK 8M 供電環 DRC 的本機重播與雙切孔

用 CBDK v2.5 8M deck 比較 SCE 的供電幾何時，若作業已在 valkyrie，可先查本機 ICV 和席位。SCE-49 的 valkyrie 與 cthulhu 使用相同的 deck 和逐 byte 相同的 ICV、manager、engine，版本為 Y-2026.03-SP1-1 cl13748545。Rocky image IDs 不同。核對現用 deck 和執行檔、CPU 與 Manager／GeometryEngine 席位，再以相同參數跑候選 GDS 一次。用封存控制版對照實跑及排除規則的全名，並把讀回 GDS、DRC 輸入與同次 DEF 的 SHA-256 綁在同一份清冊。

2 µm M6／M7／M8 間接點會觸發雙切孔規則。在偏好的線交點把 kit `VIA67_2CUT_N` 的 origin 向下位移 0.240 µm，`VIA78_2CUT_E` 的 origin 向左移 0.450 µm，可把完整 pad 與雙切孔置中於 2 µm 交點。由 2 µm M3 根接到 M6 可將 `VIA34/45/56_2CUT_N` 對位，三個 masters 共用相同的 M4/M5 landing 形狀。仍需從實際 tuple 核對包覆、同網落接與正式 DRC。

證據見 SCE-49 報告 `exp/area/safe-fp/sce49-3x8-power-overlay.md`（`3645e0d`）。
