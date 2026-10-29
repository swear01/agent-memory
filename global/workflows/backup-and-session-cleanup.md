---
title: 備份只在必要時做、驗證後清掉；HAPI session 用完必 archive
scope: global
status: active
confidence: high
source_refs:
  - user-confirmed:2026-10-30
created: 2026-10-30
updated: 2026-10-30
tags: [user-preference, backup, cleanup, hapi, sessions]
---

# 使用者偏好（2026-10-30）

- 備份只在真正需要時才做（例如不可逆的 config／binary 替換）。新狀態驗證通過後，
  刪除 backup、temp、rollback 副本，不要在各機留下一堆 `.bak` 檔。
- Agent 開的每個 HAPI session 結束後都必須 archive，收尾時再檢查 hub 是否有殘留
  （含驗證用的一次性 session）。
