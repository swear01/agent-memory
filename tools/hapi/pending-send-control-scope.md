---
title: "送出請求 pending 不應無差別鎖住模型設定"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: e65ea79f77f9856831da3735e09620fe0be9fe29a3426741c179498464c6c345
---

# 送出請求 pending 不應無差別鎖住模型設定

歷史助手發現 composer 的 isDisabled 共用 send pending 狀態，因此一個尚未返回的請求會一起鎖住獨立的模型設定；來源也指出 fetch 沒有 timeout，實際 Pi 通道是 RPC。

把送出中的請求、agent 回合及各設定可否改動分別建模，沿前端 disable 條件與實際請求生命週期核對。窄化鎖定及 timeout 是當次建議，沒有完成修復或端到端驗證；不能只由畫面卡住便斷定是 ACP 傳輸問題。
