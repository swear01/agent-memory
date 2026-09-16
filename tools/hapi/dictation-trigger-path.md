---
title: "聽寫與 Voice Assistant 的相同症狀要分路徑驗證"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: 2e6e9df1ab944739cb861a515ab48311d825c6565b0fd8ecc4bd39867d9a9a11
---

# 聽寫與 Voice Assistant 的相同症狀要分路徑驗證

Bot 驗證的是 Voice Assistant 送 agent 訊息造成的 isSending lock；助手查到 useDictation 不設定同一個 lock。使用者確認實際使用聽寫，也允許檢查兩種模式。

套用修補建議前，核對使用者的觸發模式與實際程式路徑。相似的停用控制症狀不證明共用同一原因；先修正可重現的聽寫路徑，再分開驗證其他模式。來源只到診斷與範圍更正，沒有修補成功證據。
