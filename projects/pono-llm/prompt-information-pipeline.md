---
title: "模型無法提出關係時先查 prompt 的資訊管道"
scope: projects/pono-llm
status: active
updated: 2026-09-16
evidence_digest: b871b80c9cd8f7a46ed9acdfc557ddc405d26b815aed867d41a54b7d642d97b2
---

# 模型無法提出關係時先查 prompt 的資訊管道

歷史對話中，舊 prompt 只顯示 1-bit 的 c，transition sketch 還把 state7 列了兩次，沒有呈現 8-bit 的 a、b。助手後來回報補入變數名稱、寬度、初值與依賴後，模型提出 eq(state7, state8)。這支持先檢查輸入資訊是否缺漏，不能直接把失敗歸因於模型能力。

核對來源解析、狀態選取、去重與送出的實際 prompt，確保目標關係所需的變數與 transition 資訊都有進入請求。分開驗證資訊傳遞正常、真實模型可提出候選，以及原始驗證問題確有改善；mock 或單一玩具案例不能代替後兩者。

來源後續摘要回報 unit、fake responder 與 live-model 測試，但也明示只驗證過 ab_sync.btor2，真實 HWMCC 效益未知。本次未重跑。早先未實作的計畫與後續完成自述須保留時間順序，不能拼成當前確定狀態。
