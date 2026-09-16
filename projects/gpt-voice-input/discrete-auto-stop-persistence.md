---
title: "離散 Auto Stop 設定不可用浮點精確比對"
scope: projects/gpt-voice-input
status: active
updated: 2026-09-16
evidence_digest: d2eb522813d3fb3b3b6a5684d51597410d6c90b4762fb8caa7a849cee04e6ace
---

# 離散 Auto Stop 設定不可用浮點精確比對

使用者回報並指出根因：SettingsStore 將 autoStopSeconds 存為 Float，1.8 秒讀回約為 1.799999952。RecognitionActivity 仍按此值等待約 1.8 秒，但 SettingsActivity 用 Double 的 options.indexOf 精確比對，找不到便顯示 OFF。只開設定頁不會關閉功能；在錯誤的 OFF 顯示下按 Save 才會把 OFF 寫回。

離散選項應持久化為整數毫秒或穩定 step 索引，明確區分 OFF。遷移舊 Float 時先讀取原型別並正規化，不可直接對舊 key 呼叫 getInt。此專案要求 0.0 遷移為 OFF，損壞值回到預設 1.8 秒；不能讓未知值靜默變成 OFF。

驗證須涵蓋每個選項重新開啟、舊值遷移、匯入、只開頁面不改值，以及重新開啟後儲存不誤關閉。來源是使用者的缺陷回報與修正要求，未包含後續修補或測試結果；不將它記成已發布修復。
