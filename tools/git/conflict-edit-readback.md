---
title: "衝突修復後檢查重複宣告與剩餘型別錯誤"
scope: "tools/git"
status: active
updated: 2026-09-07
evidence_digest: 00033c63ddd1efa1ad438d31b5b47c75427f9de593c8810343dda219c5c580d9
negative_result: true
redaction: passed
---

# Problem

處理 launcher 衝突時，文字替換留下重複的 cleanup 方法宣告。

# Mechanism

替換文字與衝突區域各保留一次相同宣告，編輯後沒有立即確認完整方法邊界。

# Durable rule

衝突或文字替換後讀回整段方法與相鄰邊界，再跑型別檢查；一個錯誤消失不代表其他錯誤也已解決。

# Boundary

適用於重複上下文或巢狀 finally 區域的編輯，不能只憑 patch 成功宣稱檔案可編譯。

# Verification

來源記錄承認先前編輯造成重複宣告，並顯示移除宣告的 patch；本次只核對重複宣告與移除 patch，不把局部編輯成功當成整體型別檢查通過。

# Negative result

This memory preserves a verified negative result.
