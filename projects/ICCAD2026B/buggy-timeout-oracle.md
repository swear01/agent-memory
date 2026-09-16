---
title: "Buggy timeout 不能代替預期差異 oracle"
scope: projects/ICCAD2026B
status: active
updated: 2026-09-16
evidence_digest: 9075124608e2f49584381c657ad8a758ec75b5c5572e6fae9c712e13396431b4
---

# Buggy timeout 不能代替預期差異 oracle

歷史探索回報 reference PASS、reverse buggy FAIL，但 buggy 失敗是 UVM timeout，助手診斷為 ecall 重新進入 handler 後迴圈，因此拒絕 promotion。

需證明指定缺陷時，先區分預期的可觀測差異與 harness 自身卡死。reference 通過加 buggy 逾時仍不足以證明原缺陷重現；須有符合該案例契約的 oracle。來源是單次拒絕與診斷自述，沒有修正 handler 後的驗證，也不代表所有 hang 類缺陷都無效。
