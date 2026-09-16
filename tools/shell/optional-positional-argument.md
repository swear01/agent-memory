---
title: "Strict shell wrapper 也要處理零參數呼叫"
scope: tools/shell
status: active
updated: 2026-09-16
evidence_digest: 8cf3c2a2e38ee539ea9ae73279e2b6bfdd4d32395b67e2101ca177582ccadf0c
---

# Strict shell wrapper 也要處理零參數呼叫

歷史助手定位 wrapper 啟用 set -eu 後直接讀取 $1；無參數進入 fallback 時觸發 parameter not set，尚未到達被包裝的 Agent。

對可省略的位置參數先檢查數量，或採明確預設展開如 ${1:-}，再測零參數與正常參數路徑。區分 wrapper 的診斷與底層工具狀態；來源只提出修法，沒有修正後實際執行證據。
