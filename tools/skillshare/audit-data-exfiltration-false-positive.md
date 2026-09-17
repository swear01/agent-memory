---
title: "Skillshare audit 的 DNS 洩漏規則會誤判路徑旁的省略號"
scope: tools/skillshare
status: active
updated: 2026-09-17
---

# Skillshare audit 的 DNS 洩漏規則會誤判路徑旁的省略號

在執行 `skillshare audit` 靜態掃描時，規則 `data-exfiltration-3`（偵測疑似透過編碼子網域進行 DNS 資料外洩）會對特定文字樣式產生誤判。

## 觸發現象
在 Markdown 說明中，若於反引號代碼塊內包含 Windows 磁碟代號緊鄰省略號（例如含冒號、反斜線與點號的省略符號字串），靜態分析器會判定為 HIGH 嚴重度的 `data-exfiltration` 警報，導致 `riskScore` 飆升（如 15 分）且被標記為 high risk。

## 預防與修正方式
1. **避免硬編碼假路徑範例**：撰寫跨平台路徑規則時，改用語意變數（如 `$REPO_ROOT`、工作區相對路徑）。
2. **文字形式敘述**：將「不可使用絕對路徑」以文字概念描述，避免在代碼字串中使用連續點或省略號相連的磁碟符號。
3. **驗證稽核結果**：提交 skill 前務必執行 `skillshare audit <skill-name> --json`，確保掃描結果達到 `riskScore: 0`、`riskLabel: "clean"`。
