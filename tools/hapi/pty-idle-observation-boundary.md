---
title: "PTY 沒輸出不代表 agent 已完成"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: a07832d0acf1e742b0e17cbfbcf0d7b8c199a2f4a65a26a4f17e0870c73b4412
---

# PTY 沒輸出不代表 agent 已完成

歷史助手指出 agy busy 判斷依賴 Generating 字串，三秒無輸出會回報 idle；使用者要求保留 PTY 方式，不採 hook／transcript 整合。

在該限制內核對真實 idle prompt、process exit 與跨 chunk／ANSI 的 marker 處理，將安靜推理與完成分開。以送出訊息設 busy、明確 idle 才解除只是當次提案，未重畫 prompt 仍可能卡住。來源沒有實作或實機驗證，不稱它已可靠取代結構化事件。
