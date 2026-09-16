---
title: "IME 與 Activity 的圖示 tint 要分別驗證"
scope: projects/gpt-voice-input
status: active
updated: 2026-09-16
evidence_digest: c397526ba7135ab6a72cc40c2861a282c5db0e979aaa2a750bf4835abcb3356f
---

# IME 與 Activity 的圖示 tint 要分別驗證

使用者更新後仍回報 IME 麥克風圖示黑色；助手診斷該 service 的 layout inflater 沒有處理原先使用的 AppCompat tint，並提出 framework tint 修法。

核對實際 View 類型、inflater、資源與深淺主題，再在 IME 真實入口看結果。Activity 顯示正常不能替 service 介面驗收，也不能以屬性名稱判所有 inflater 行為。

這是歷史診斷假設與建議，來源沒有 IME 視覺修後回讀；設定頁黑畫面的 force-dark 假設與後來 CI runtime 升級分開處理。
