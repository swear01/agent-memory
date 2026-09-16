---
title: "累計 token 的重設邊界不能只看 session ID"
scope: tools/observability
status: active
updated: 2026-09-16
evidence_digest: fca23a93aba8178999205569c2b76bfd19af5fc3fbe115b31404e3fc527e94a4
---

# 累計 token 的重設邊界不能只看 session ID

歷史助手發現同一 session 的續寫 rollout 檔重新起算累計值；原本跨檔追蹤同一 session 並跳過負 delta，造成續寫檔少計。另回報直接加總 last_token_usage 會超過檔內最後 total。

先驗證該格式的計數範圍與 rollover 行為，再處理 delta、重複事件與檔案邊界；用檔內 final total 交叉核對。來源是歷史診斷與自述數字，未完成外部工具演算法確認，不能把每檔重設或增量欄位語意套用到所有 provider。
