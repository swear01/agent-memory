---
title: impeccable 決策頁在遠端主機無法給使用者開啟
scope: tool
tool: impeccable
tags: [impeccable, serve-question, oracle, remote]
status: active
updated: 2026-10-20
---

`impeccable serve-question --start` 會在本機另開一個 port 給使用者選方向。在 Oracle 主機上，公網只開放 22／43005／53346（見 `machines/oracle/vaultwarden.md`），而 53346 保留給遊戲 playtest，使用者從外面打不開決策頁。

做法：不要啟動頁面；直接用結構化問題工具呈現 assigned／pick／challenger／canon 選項，並在第一次回覆就說明改用這個替代方式。選定後照常用 `concept-seed --from <key> --kind <...> --chosen <SOURCE ID>` 回報選擇。

另外：impeccable 要求的 finish reviewer 與 documenter 在 Claude Code 沒有專用 agent type 時，可用 general-purpose subagent 載入 `reference/degraded/finish-reviewer.md`／`documenter.md` 來跑；reviewer 必須是全新 context，並把截圖路徑、方向契約與 PRODUCT.md 一起交給它。
