---
title: "評測前核對選集，不能跑完才發現用了排除集"
scope: projects/cpachecker
status: active
updated: 2026-09-16
evidence_digest: 1485d42db2e5659f587f6efb493e605be3878317402fa7b6825e1a7b4d5220c2
---

# 評測前核對選集，不能跑完才發現用了排除集

使用者要求 non-trivial，歷史助手卻跑了 excluded_trivial_solved 的四十四題；被糾正後才回報啟動指定的四十題 non-trivial 選集並另設輸出目錄。

啟動前核對資料集用途、完整成員與指定選集，題數只作輔助。選錯集合的結果保留為其本身範圍，不能當成要求中的主要評測。來源只顯示修正後啟動，沒有 non-trivial 完成結果。
