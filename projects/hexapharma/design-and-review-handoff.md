---
title: HexaPharma 視覺方向與玩法審查交接
scope: project
project: hexapharma
status: active
updated: 2026-10-21
---

- **復用舊極簡 pixel 方向或讓畫面比工廠更難讀。** 採定向越野圖的紙底墨印與 ISOM 色語，保留現代簡潔線條和手繪感；路線紫只屬玩家的研究／候選／選取／Plan。桌機優先、手機可操作，port、flow、數字及速度優先；見 repo `PRODUCT.md`、`DESIGN.md` 及 surface contract。
- **白箱產線能出第一份藥，後續卻不夠錢或沒完成合約。** `npm run progression` 需核真實 shippingContracts、每病 net 與最低現金；用完整多 seed ledger，避免只降專利價貼合最早幾題。Reference/compiler 先預留 input approaches 與 output exits，避開較早 BFS 皮帶堵住後機出口。第一份合約後的指示核對 active disease 的 cure；已有機器不代表目前產線正確。白箱可解不證新玩家理解或好玩。
- **Open save 改了合法資源卻被驗證拒絕，或 live run 到累計 ticks 停止。** 使用者要求明文可編輯存檔，驗結構、大小、安全整數、局部一致性和單次工作量，不驗資源來源。v11 用冷狀態而非終生 replay；Recover 可在有界外層資料救回各別合法快照，但保留損壞項缺口，不跨缺口連歷史。Deadlocked 診斷每 tick 重算，合法週期 source 不以靜態旗標永久停。以實際 latest Research 順序核 `discoveredFormulas`，避免只顯示最後一病或指到舊配方。
- **手機 cover／modal 或 HMR 造成假驗收。** Camera clamp 用真實裁切 viewport，zoom anchor 扣中心 offset；手機 Formulas 遮 world 時與 Factory／Research global keys 共用 visibility guard，world input 分別管 timer。Escape drawer 先於 focused field guard，blocking dialog 再優先。用 1280px／390px 地圖角落與 Play 中多跨 intervals 驗。跑 `npm run check` 不改同一 worktree，平行 E2E 設 `E2E_DEV_PORT`／`E2E_PREVIEW_PORT`；截圖等 Pixi canvas 真的出畫及 toast 消失，Cure camera fit 要實際完成最後一步。
- **公開 Pages 的程式已合併，網站仍不可用。** 主站與 Pages 子路徑同用 build `--base=./`，只在 main check workflow 成功後部署；存檔在瀏覽器。若 HTTPS provisioning 卡住，先讀 Cloudflare DNS/CAA／Pages ref 與錯誤，不改另有網站的根域。原修復是 Pages 的 cname 重新綁定後才可開 HTTPS enforcement。Bot 指成 v5 action 不存在時，用 `gh api repos/<action>/git/ref/tags/v5` 查真正 tag；多頁 review 用 `gh api --paginate` 及 `original_commit_id`。

[原始紀錄](https://github.com/swear01/agent-memory/blob/9b2d556e370f3a71736d5dc0a63f3b1e1c8c3a0e/projects/hexapharma/design-and-review-handoff.md)
