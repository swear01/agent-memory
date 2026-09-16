---
title: "Save 的局部一致性驗證不是防篡改保證"
scope: projects/hexapharma
status: active
updated: 2026-09-16
evidence_digest: 046b8ad29c074bc10b5ea64e515dbc0c9087fd8a2393aa99b064eccf4d70098d
---

# Save 的局部一致性驗證不是防篡改保證

歷史唯讀稽核回報 deserialize 接受降低 cost／progress 及複製有效庫存後換 ID，表示 schema、範圍與唯一性檢查沒有驗證生產 provenance。

按實際保證收窄文件；若要拒絕這類偽造，須另證完整來源／replay 契約，不能靠非負與 ID 唯一性宣稱 tamper-proof。来源另列未 tracked baseline、跨 level sold、UI 揭露等獨立問題，本筆記不把它們合成存檔根因。當時未改檔，也沒有修復後驗證。
