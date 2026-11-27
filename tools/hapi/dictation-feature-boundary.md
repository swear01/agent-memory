---
title: HAPI Dictation 語言或送出失敗，先核對實際觸發路徑
scope: tools/hapi
status: active
updated: 2026-11-28
---

聽寫語言不符或控制停用時，先辨認是 Dictation 還是 Voice Assistant，再核對 provider、標準／即時模式、hook、route、送出的語言欄位與回傳文字。已核對的 Voice Assistant `isSending` lock 沒在當時的 `useDictation` 設定。只在 Voice Assistant 重現的結果無法驗證 Dictation。語言提示與繁簡輸出要用實際請求及回覆驗證。

Connected Dictation 的鍵盤 Enter、滑鼠與觸控應能執行已定義的 Send。保留 Stop 原有語意，驗證每種輸入各自能完成 Stop／Send。原始紀錄涵蓋診斷與 finding，相關提案的修復結果須另查。

[原始路徑診斷](../../archive/tools/hapi/dictation-trigger-path.txt) · [輸入方式 finding](../../archive/tools/hapi/dictation-send-input-parity.txt)
