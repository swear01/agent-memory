---
title: "Provider 參數的位置不能靠相似名稱猜測"
scope: tools/api
status: active
updated: 2026-09-16
evidence_digest: 19677bd45246dfc6d5f17ee1bd610fbbce9cfa2c95d2e3c45f187bebc6f41910
---

# Provider 參數的位置不能靠相似名稱猜測

歷史助手先嘗試把 thinking 設定放入 extra_body，遭拒後又提出另一種 wrapper；使用者提供當時相容端點的文件，指出 reasoning_effort 應在 create 呼叫的頂層，助手才更正映射。

先對照實際端點與客戶端版本的請求 schema，分開標準參數和 provider 額外欄位，不把同名功能的不同協定格式混用。来源沒有更正後請求成功的證據；當時允許值與 wrapper 格式不能泛化到所有 provider。相鄰 dotenv 載入順序故障是另一個原因。
