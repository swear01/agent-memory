---
title: "移除子命令後要檢查未知命令的預設路徑"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: 20ad518d67d15ab0664b46c29ee0573716209271f180f223b83e12d0b15e3d52
---

# 移除子命令後要檢查未知命令的預設路徑

歷史 bot 指出刪除 geminiCommand 後，hapi gemini 被 resolveCommand 當成預設 Claude 命令，將 gemini 轉成參數；來源顯示隨後提交並推送明確拒絕的 tombstone。

命令退役要驗證解析後的實際行為，避免原指令轉而啟動另一個 agent。可在該解析契約保留明確退役錯誤，或由共用未知命令處理拒絕；不把 tombstone 當所有 CLI 的必要架構。來源沒有最終執行測試或合併結果。
