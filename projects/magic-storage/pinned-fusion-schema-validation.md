---
title: "資源測試要對齊實際部署的 Fusion schema"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: 690408ca2fb2a84000dee509d3e6735c65d9d66156bdf1c5fd69bf7148007dea
---

# 資源測試要對齊實際部署的 Fusion schema

歷史只讀審查回報七項 unit tests 全綠，卻要求釘選 Fusion 舊版不支援的 blocks 與 false predicate；臨時來源及範例 checkout 已是新版。相同審查文字重複出現，不算兩次獨立驗證。

以實際 resolved artifact 的 parser、生成 metadata 與 client resource reload 核對相容性，不讓測試鎖住錯誤新版假設。build dry-run、JSON 外形及下載 hash 各有不同責任；此來源沒有啟動 client，視覺、classpath 與 rollback 完整性不能一併稱通過。
