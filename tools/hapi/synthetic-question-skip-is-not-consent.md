---
title: "Headless 自動跳過問題不是使用者同意"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: d18b0c650db22efc754966dea98d2f594ff92ac224bf47f60827f771596e5103
---

# Headless 自動跳過問題不是使用者同意

保存 issue body 指出 headless cursor-agent 在沒有 IDE 問卷介面時立即回傳 questions skipped 且 is_error=false，agent 因而誤以為使用者跳過。該紀錄列出的後續呼叫是只讀，破壞性後果只是風險推演。

核對回答是否來自可用的人機介面與真實 operator；工具成功旗標、合成 skip 或沉默都不能創造新的授權。來源僅是歷史 issue 報告且本文截斷，CLOSED 狀態沒有修補或部署證據，也不能把自述全重現率外推所有 CLI 版本。
