---
title: "更新自動 review 流程要一併清除跨 skill 舊指令"
scope: tools/agent-rules
status: active
updated: 2026-09-16
evidence_digest: 4b8eb6a4c2174171c7c63c4645c91a2e9eb488d76e9051c967f7163545a4902f
---

# 更新自動 review 流程要一併清除跨 skill 舊指令

歷史檢查找到 review skill 已寫 push 自動觸發，preconditions 與另一份 PR workflow 卻仍要求所有 provider 手動觸發；同步版本與 canonical 相同，問題是內容互相矛盾。

依已驗證的 provider 行為修正所有受影響入口，區分自動觸發、手動補救與可選額外 review。使用者允許有需要時增加審查，不代表每次強制多個 provider，也不能只靠同步成功宣稱規則一致。來源沒有後續修改發布證據，歷史設定不代替當前實際狀態。
