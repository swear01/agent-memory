---
title: "驗收項目必須真的存在於 GUI fixture"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: eb5001d785aa7bd2c3d022cace49eedc4064a3ab609eb361f9a21855a1913ed9
---

# 驗收項目必須真的存在於 GUI fixture

使用者指出 deterministic Prism GUI lab 的 gallery 與 active network 只有 T1–T6，沒有 creative_storage_unit，因此無法執行已列入清單的 Creative Unit 貼圖與終端無限容量驗收。

交付測試環境前，把驗收清單逐項對到實際放置的元件、連線與必要狀態；補齊 fixture 與其自動檢查後，手動驗收仍須另列。此來源明確要求當次只改自動化 fixture、不啟動 Prism；沒有後續修補結果，不能記成 fullscreen gate 已通過。
