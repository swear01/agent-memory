---
title: Save Design Compiler mapped artifacts before evidence Tcl
scope: domains/eda
status: active
updated: 2026-11-28
---

# Design Compiler 合成後先寫 netlist

一次限定每臂只合成一次的 matched DC 對照中，基準臂完成映射並寫出原生 timing 和 area 報表。後續 Tcl 假設每個 leaf cell 都有 `area` 屬性，遇到 UID-101 後中止。DC process 回傳 0，產物檢查才發現 mapped Verilog 缺失。工作目錄只剩分析庫和 SVF，無法從這些產物重新取得該次的 netlist hash。

- 最後一次 compile 結束後先寫 mapped Verilog。若需要重新產生 timing 報表，保留帶約束的 mapped checkpoint，讓報表可以脫離合成重跑。
- 原生 area 報表可作共同面積口徑。自訂逐 cell 統計遇到缺失屬性時記錄 reference，再由原生報表和設計檢查釐清。
- Runner 同時驗完成標誌、netlist 檔案和 checksum。process exit code 也要保留，便於區分合成、Tcl 報表和產物檢查各自在哪一步結束。
