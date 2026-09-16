---
title: "聯合搜尋測試須正確對照成功路線與失敗基準"
scope: projects/hexapharma
status: active
updated: 2026-09-16
evidence_digest: 29b39ccfa7f7caa577866af49df9b4f502f398f1d53415685550dce2e70bfe2f
---

# 聯合搜尋測試須正確對照成功路線與失敗基準

保存的輸出中 naive 三步路線為 failed:true，solver 的另一條三步路線為 failed:false；草稿曾把兩者成敗寫反。

把每條路線的輸入、機器序列與 outcome 綁在一起比較。要證明聯合搜尋避開單一路線的陷阱，測試必須確認基準失敗、solver 成功，而且確實用了不同路線。

來源是使用者貼出的輸出與助手解讀，本次沒有重跑遊戲。不能僅憑兩條路線同為三步就判為等價。
