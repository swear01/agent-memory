---
title: CVSD SCE 的 CTS 面積抵扣按同一次 pin graph 取聯集
scope: projects/cvsd-final-sce
status: active
updated: 2026-11-28
tags: [sce, cts, area, reachability]
---

比舊 postCTS 面積時，後續 route/export 上同名的 instance 或面積不能直接拿來抵扣。同一次 CTS checkpoint 可從 `SCE.symtbl.gz` 核實 instance/master，從 `vbin/SCE.v.bin_0` 和 `SCE.v.bin_lib` 讀回 pin graph，再用該 checkpoint 的 LEF 作 cell 價格。SCE-47 的只讀解碼先對清冊、驅動和 net 完整性，再從所有 IMG pins、頂層 I/O 和保留 sequentials 逆追。各次 cut 用確切 instance 聯集，混合消費端留在帳內，新端點接管另付款。

立即寫回固定段可以淘汰舊批次 T/shift/threshold producer，但 rolling M/U、`iqr`、`cwr` 和 `g_prv` 要按實際讀者分類。舊 T 的 direct FF 面積已含在整條排他寫入 cone，不能再加一遍。跨場景分配 byte 槽時要在消費前、Q 已接受後量峰，通常還需帳列地址標籤和延後退休的 fork。SCE-47 的全零對照會越過公版估出的槽位，單一公版峰不可當合法容量上界。

[同次 checkpoint 重數工具與 SCE-47 證據](https://github.com/swear01/cvsd-final-sce/blob/c0c0e766229dfaac38de695a4264797ce231e267/exp/area/segmented-overlap/sce47-state-budget.md)
