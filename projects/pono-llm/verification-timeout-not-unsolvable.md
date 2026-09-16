---
title: "候選驗證逾時不能寫成等式不可解"
scope: projects/pono-llm
status: active
updated: 2026-09-16
evidence_digest: cae9b83e62babbe7605e9a73d1719e985534155cdb33b283c552c4850d383a9a
---

# 候選驗證逾時不能寫成等式不可解

保存日誌顯示二次等式在 round 2 因 verification timeout 被拒；另有不等式被標為 SOUND，以及不同候選因不支援 implies 而失敗。助手把逾時描述成不可解，超出輸出所能證明。

分開記錄 timeout、unknown、語法不支援與已取得的證明結果。依總預算決定重試，不把逾時當反例，也不把某個替代式成功推成原式不成立。

本次未重跑 solver 或獨立驗證 SOUND 標籤。縮短 timeout 只是原對話的計畫；fib_30 的總時間不能誤寫成全部由 preprocess 花費。
