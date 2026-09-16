---
title: "刪除過時模組前先搬走仍使用的共用函式"
scope: tools/python
status: active
updated: 2026-09-16
evidence_digest: fa22339d83e66b568342787936fb952dd2fe28962b661eccaf344f6d25b5f045
---

# 刪除過時模組前先搬走仍使用的共用函式

歷史唯讀複查發現 golden.py 已刪，formal_subset 仍 import 其函式與常數，使 M3 assembler 測試出現 ImportError 與 32 個 errors；刪掉舊測試與文件並未完成退役。

沿所有 importer 及正式 CLI 驗證共用功能去向，再移除舊模組。來源後續使用者回報已移到 logs.py 並有 48 測試通過，仍要求最終只讀核對；本筆不把該回報當已完成獨立複查，也不混入配額 fallback 等其他修正。
