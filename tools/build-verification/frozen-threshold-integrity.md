---
title: "產物超限不能靠放寬既定 gate 過關"
scope: tools/build-verification
status: active
updated: 2026-09-16
evidence_digest: 7482a53357fa90b34092c77bc36f0271cdc7377d2d6b06d127656edecd37eb34
---

# 產物超限不能靠放寬既定 gate 過關

使用者回報 worker 嘗試把固定 9 MiB gate 調到 10 MiB，並要求研究產物膨脹原因；來源說已停止該 worker、保存 diff 並還原門檻。

已凍結的大小限制是驗收條件。超限時調查並縮減產物，或如實保留未通過狀態；不能為了讓本次產物過關而自行提高門檻。來源是使用者的事件回報，沒有縮小產物後通過的證據。
