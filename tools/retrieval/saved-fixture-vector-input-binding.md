---
title: "重播保存向量時案例 ID 一致仍不足以證明輸入相同"
scope: tools/retrieval
status: active
updated: 2026-09-16
---

# 重播保存向量時案例 ID 一致仍不足以證明輸入相同

本次 G16 接續漏查了下游案例路徑：腳本拿舊 G3 fixture 的 query 文字，搭配已通過 G11 修正版的向量與反向輸入。五個案例 ID 和 expectedContentHashes 都未變，但 query、formattedText、token hash 等已改；只按 ID 比較正反序向量無法攔截這種錯配。上游報告通過，不代表下游沿用了同一份輸入。

從已通過關卡保存的 amendment 與 artifact hashes 綁定實際 query pack、向量及反向輸入。恢復前比較正向列與反向還原列的完整欄位，只排除因反轉而改寫的 ordinal；本次檢查涵蓋 48 個查詢和 5 個案例，正確保存輸入通過，舊版錯配輸入在啟動前被拒絕。不要只修路徑而缺少這個負向檢查。

本次有意停止錯配的檢索子程序，原有 rollback 通過，保留原腳本、兩版輸入、向量、日誌與兩份已成功的唯讀報告。核對所有前綴雜湊及資料庫身份、大小、修改時間、空 WAL 後，只恢復未完成的檢索步驟；最終仍要求完整結果與保存 G11 報告逐位元組相同。這不是重新產生 golden expected hashes，也不能把修正後的來源案例當成原本廣泛問題或全語料召回的證明。
