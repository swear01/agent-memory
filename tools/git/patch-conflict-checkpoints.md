---
title: "連續套 patch 必須先停在尚未解決的衝突"
scope: tools/git
status: active
updated: 2026-09-16
evidence_digest: 32432d56bce8391919159967c55dfad2bcea580b652e6b044502c44e01186ee7
---

# 連續套 patch 必須先停在尚未解決的衝突

歷史助手承認迴圈未在 patch 之間解衝突，後續 apply 疊在未解決狀態上。改用舊 release 的解析檔時，又遇到 upstream delta 與既有 carry 內容無法直接疊加。

每個 patch 完成後確認衝突已解並保存可恢復狀態，再處理下一個。重用舊解析必須比對完整基底、carry 與本次上游變更，不能只看檔名相同。來源沒有最終驗證結果，也不提供丟棄現有工作的授權。
