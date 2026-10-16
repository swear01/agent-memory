---
title: PHASE HTML demo 最終版 v10
scope: projects/phase-html
status: final-demo
updated: 2026-10-16
tags: [PHASE, phase-demo, HTML, demo-final, Sites, GitHub, itch.io, tile-grid, pixel-art]
---

# PHASE HTML demo 最終版

2026-10-16 使用者確認目前公開遊戲是這個 HTML demo 的最終版。保留此版本作為基準，除非使用者提出新需求，不再主動擴充關卡、機制或美術。Gamejam1 Unity 專案的狀態另見該專案記憶。

## 定版來源

- 公開網址：<https://phase-three-color.swear01.chatgpt.site/>；Sites 第 **10** 版，audience 為 public，部署成功。
- 沿用 Sites project ID：`appgprj_6acf3b6b27d08191ad352a85e3e14d70`，不可另建站台。
- 開發 commit：`e88de0563d9dc4b41cc25c83d5d204dce9c957e8`；branch `fix/phase-inactive-spikes`；工作樹 `<worktree-root>/phase-inactive-spikes-20261016`。開發 Git 未設 remote，舊 `main` 不是定版來源。
- 已推送的 Sites source commit：`d34a051b7ea157e7bc8ac845fba5c3c9bc7e24e1`，靜態檔是 `dist/index.html`。
- HTML SHA-256：`64f242b932ffe3803eae744665ca8447ccf098a740bdf1890db6ab0481b61b58`。
- 工作區 `<codex-root>/2026-10-14/html-2-thread-01a13540-1e2a-7163`：離線檔 `outputs/phase-three-color.html` 與發布原始碼一致；核心、地圖、美術說明在 `outputs/phase-core.md`、`phase-level-design.md`、`phase-art.md`；發布與驗證收據在 `work/phase-inactive-spikes-publication.json`。

## phase-demo 公開封存與 itch.io

- 使用者於 2026-10-16 指定保存為 `phase-demo`，GitHub 必須 **public + archived**。
- 公開封存 repository：<https://github.com/swear01/phase-demo>。已於首次推送前完成 `gemini-link`，完整保存 73 個專案與定版說明檔。
- 封存 commit：`2061d8e938ce88e3d2ec468c44e6fe98a33626ba`；定版標籤與 Release：<https://github.com/swear01/phase-demo/releases/tag/demo-final-v10>。
- Release 附 `phase-demo-final-v10.zip`，19,959 bytes；ZIP SHA-256 `0e6b863914bb1ce7a2698cbcb9bb779c8960e126cb773c538c424a53b01a38ff`。匿名下載解壓後，`index.html` 與上列定版雜湊一致。
- itch.io：<https://swear01.itch.io/phase-demo>，game ID `5168384`，顯示 Published；免費且不收款、桌面鍵盤、960×640 嵌入、全螢幕按鈕、封面與三張遊戲截圖，已按實際內容填寫 AI disclosure。
- itch.io HTML upload ID `19800447`；匿名遊戲檔 HTTP 200，engine／CSS／renderer 與最終版一致。匿名公開頁曾暫時回傳 429，後恢復 200，實際頁面內 iframe 的八關 60 Hz 鍵盤通關、換色／衝刺、暫停、重玩與死亡重生檢查通過。
- 原 Sites 公開網址保留。完整發佈收據為工作區 `work/phase-demo-publication.json`；`outputs/phase-demo-final-v10.zip` 與 `phase-demo-itch-public.png` 保存離線包和頁面證據。
- itch.io 介紹的 HTML 編輯器只用 `fill()` 再切回富文字時，預覽曾有內容但儲存後為空。輸入後送出 Space、Backspace、Tab，確認具名 `game[description]` textarea 已同步，再儲存並讀回公開段落；不要把編輯器預覽當成已保存。

## 定版內容

- 電腦鍵盤遊戲，無虛擬操作按鈕；原生 320×180、地形 8×8、每房 40×23 格，底列顯示 4 像素。`levels.json` 為地圖來源。
- 成功跳躍與牆跳依序紅 → 藍 → 綠 → 紅；無效空中跳不換色，衝刺、攀爬、自動翻牆保色。保留既有 Celeste 風格物理、0.1 秒離地寬限與 0.08 秒預輸入。
- 七種平台與尖刺相位：紅、藍、綠；黃＝紅＋綠，紫＝紅＋藍，青＝藍＋綠；白永久啟用。啟用尖刺致命，停用可穿過；換色令身上尖刺啟用時立即死亡。
- 八關：紅刺、頭頂、保藍、色帶、雙門、五跳、換色台、歸途。以三面牆五連跳的借力與換色規劃為基準；八關目前未使用移動平台。
- 原創像素美術由本機 Claude Code `claude-opus-5-5` 重做：Wick 兜帽旅人、跑跳衝刺攀爬姿勢、連接石磚、石拱出口與月夜背景。眼睛表示相位，琥珀／灰圍巾表示衝刺是否可用。
- 使用者接受的尖刺 **A 方案**：停用時是低彩度暗色實心齒形，保留朝向與範圍；取消空心線框與點狀底線，混色標記只留整串第一格。啟用外觀與判定不變，換色當幀切換顯示。
- 相接的左右入口使用滑動轉場；同側、內部出口或高度不符時短暫遮黑。維持單一公開遊戲分頁，更新時重載原頁。

## 驗證邊界

- 定版當天，本機八關 60／120 Hz 鍵盤通關通過；匿名公開頁 HTTP 200、八關 60 Hz 通關及 engine／CSS／renderer 與已審閱原始碼一致。
- A 方案確認 28 張啟用尖刺素材像素不變、四方向停用齒形實心、混色標記位置、相位快取立即切換；物理、地圖、角色與其他繪圖區塊未改。
- 先前保留 40 組獨立移動／衝刺／攀爬參考軌跡、31 條完整容錯路線與 13 條啟用尖刺失誤路線。有限搜尋與腳本通關不能證明唯一解或真人難度；新版尖刺辨識度尚未做真人測試。
- 若另有更新需求，從上述定版原始碼接續，沿用現有 Sites 身分，完成來源推送、保存版本、部署與匿名檢查後再聲稱公開版已更新。
