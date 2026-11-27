---
title: PHASE HTML demo 最終版 v10
scope: projects/phase-html
status: final-demo
updated: 2026-10-16
tags: [PHASE, phase-demo, HTML, demo-final, Sites, GitHub, itch.io, tile-grid, pixel-art]
---

- **把 HTML 定版當 Unity Gamejam1，或發布時開第二個站。** 使用者要求保留 HTML v10 與原遊戲身分，先取 [公開封存的 demo-final-v10 release](https://github.com/swear01/phase-demo/releases/tag/demo-final-v10)。實際 Sites source 是 `d34a051b7ea157e7bc8ac845fba5c3c9bc7e24e1:dist/index.html`，開發舊 main 不作定版來源。公開 [Sites](https://phase-three-color.swear01.chatgpt.site/) 與 [itch phase-demo](https://swear01.itch.io/phase-demo) 均需匿名讀回，不因短暫 429 換成另一遊戲。除非使用者提出新更新，維持歷史定版。
- **itch 富文字預覽有介紹，但存完為空。** 送 Space、Backspace、Tab 讓具名 `game[description]` textarea 同步，再儲存與讀回公開段落。對公開遊戲檔核正確 hash 和真實鍵盤操作；腳本八關通過只證指定路徑可玩，不能保證難度或唯一解。
- **修改 inactive spikes 牽動了碰撞或色相。** 定版停用齒形是低彩實心，保留方向與範圍，混色標記只留串首。跳或 wall jump 才依序換相，失敗空中跳及 dash/climb 不換相；換相當下的內嵌尖刺判定要維持。更新需重新核啟用尖刺像素、四向停用樣式和原生流程。

[原始紀錄](https://github.com/swear01/agent-memory/blob/9b2d556e370f3a71736d5dc0a63f3b1e1c8c3a0e/projects/phase-html/final-demo.md)
