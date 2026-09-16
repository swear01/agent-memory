---
title: "未觀察到 client 事件不能直接判定不支援"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: 58afa872fbe6a0f062b163264eb94f669f4b61ec823e57ee04caaad1e59fa66b
---

# 未觀察到 client 事件不能直接判定不支援

歷史助手只對一個 ACP client 回報標題事件實測，其餘因未觀察到或未安裝而保留 fallback；使用者要求查官方資料，並明確不要 fallback。

將標準支援、特定版本實作與本次事件觀測分開核對。缺少單次事件只是線索，不足以判定能力不存在；是否新增 fallback 仍依當次需求。來源沒有後續查證或移除結果，另提到的平行測試衝突也未證明與改動完全無關。
