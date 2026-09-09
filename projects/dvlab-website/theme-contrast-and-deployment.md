---
title: DVLab 網站深淺色對比、粒子配色與主站部署驗證
scope: projects/dvlab-website
project: DVLab-NTU/dvlab-ntu.github.io
status: active
updated: 2026-09-09
---

## 主站與備援

- 主站為 `https://dvlab.ee.ntu.edu.tw/`，由 Inari 的 Caddy 提供靜態網站；`https://dvlab-ntu.github.io/` 是可直接瀏覽的 GitHub Pages 備援。
- 兩站都以 `PUBLIC_SITE_URL=https://dvlab.ee.ntu.edu.tw` 建置 canonical、hreflang、Open Graph 與 sitemap。不要將主站綁到 GitHub Pages，也不要讓備援重新導向主站。
- GitHub Pages 隨 main 更新，Inari 則需要獨立部署。PR 合併／Pages 成功不能證明主站已更新。
- Inari 的 `/Library/WebServer/DVLab/current` 指向 `releases/` 內的版本；部署應新建 release、保留舊版，核對目前版本後原子切換 symlink，不需重啟 Caddy。
- 2026-09-09 早先 SSH 的 `sudo -n true` 需要密碼；19:20 後重新實測已成功，管理員障礙已解除，未來仍先重驗。若之後權限不足，可先將已驗證的封裝與部署腳本放在 `<remote-home>/dvlab-deploy-<commit>/`，核對 SHA-256、路徑安全、canonical 與預期舊版，再請使用者以管理員執行切換。不得將「上傳／預檢成功」記為「上線」。

## 淺色文字根因與修正

- PR #87：`https://github.com/DVLab-NTU/dvlab-ntu.github.io/pull/87`；合併提交 `5c9b7330a3e4d0ded1cf27df46f6bd73d58af852`。
- `src/styles/components.css` 的 `.badge` 使用淺色 `--primary-2`，淺色模式下與淡黃卡片背景幾乎相同；包含 ICCAD 2010 的所有論文會議／期刊，以及成員身分，皆受共用模板影響。
- 淺色 `.badge` 改用既有 `--text-soft`；頁尾連結 hover 改用 `--text`；導覽列使用深色字並取消淺色模式的整體 opacity，active／hover／focus-visible 用 `--text`。
- 不要直接將 `--primary-2` 全域改深：它同時用於按鈕背景與深底上的淺色字。應在用途相同的共用元件修正，保留深色模式。
- 43 篇論文與 19 位成員，中英文共 124 個詳細頁；badge 對比度從 1.02:1 提升至 8.75:1。這是當次資料規模，內容增減後應重新統計。

## 驗證方法與易漏事項

- `npm run verify` 在 Node 22.16.0、CMS disabled／enabled 設定都通過；CMS enabled 測試使用 example OAuth URL，只測產物生成，不能宣稱真實登入成功。
- `npm run test:browser` 已接入 `tests/contrast-browser.mjs`：量測淺色文字至少 4.5:1，含兩語言、390／1280px、normal／hover／keyboard focus；另確認深色 badge 字色未變。先對未修正式站執行，確認測試能抓到 1.02:1，再對修正版驗證。
- 自動對比掃描須合成半透明背景及文字 opacity；漸層和照片背景不能只依 background-color 下結論，應標記待人工／像素驗證。深色主按鈕忽略黃色漸層時會出現假的 1:1 結果。
- 瀏覽器測試不可與清空／重建同一份 dist 的工作同時執行；曾因此看到暫時性首頁連結 404，建置結束後重跑完整瀏覽器測試通過。
- 不要將一次明示的「略過機器人審查」保存成永久偏好。本次 PR #87 的略過及合併已獲使用者明確同意。

## PR #87 歷史驗證（後續版本見下節）

- 2026-09-09 主站已觀察到 `current -> releases/20260909-5c9b733`；先前「等待管理員切換」狀態已過期。
- GitHub Pages 與主站各完成 137 個 URL 的深淺色掃描，無瀏覽錯誤，124 個詳細頁 badge 都是修正後的 8.75:1。主站 HTTPS、英文論文深層路由及不存在路由的 404 狀態亦確認。
- 本任務預備封裝（非實際啟用封裝的身分證明）SHA-256 為 `619dc7a762cd8c2dc212d6d65e5b7cf28eb0ffec5e59f4dd7ef2003ad108ddf2`；未保存任何密碼或憑證。
- 已清理本任務乾淨 worktree／分支並快轉本機 main；其他任務 worktree 保留。

## 首頁粒子效果與可見度

- 舊版 `6ff739c` 的 `frontend/src/components/Home/index.js` 使用 react-tsparticles：淡黃 `#fcffcc` 方形粒子、粒子 opacity 0.5、連線距離 150px／最高 opacity 0.1／寬 0.5px、滑鼠 repulse 距離 200px、點擊 push 四顆。
- 搜尋前端特效時必須包含 `.mjs`：新版已有 `src/scripts/particles.mjs`，不能因只搜尋 js/ts/astro/css 就判定沒有實作。首頁重構移除了 canvas 與載入點；PR #88 接回 `HomePage.astro`。
- 特效限中英文首頁的大 Logo 開場背景，是粒子與連線受到游標排斥，並非游標換形或拖尾。裝飾 canvas 不攔截事件，背景容器接收 pointer 事件，Logo／連結點擊不新增粒子。
- PR #88 的動畫測試通過，但使用者仍看不到：RGBA token alpha 又乘上 canvas globalAlpha，使淺色粒子只剩 7.5–17.5% 不透明度、連線最高 2.5%。只證明有 draw calls 不夠；須以實際尺寸、兩種主題目視確認。
- PR #89 改為不含 alpha 的色彩 token，透明度只套一次：深色 `--particle-color: #fcffcc`、粒子 0.5、連線最高 0.1；淺色採 `#526326` 深橄欖綠、粒子 0.5、連線最高 0.22，搭配原有黃綠背景。方形大小為 2–6 CSS pixels，排斥距離恢復 200px。
- 原生 canvas 移植保留舊版視覺元素與主要互動，不宣稱完整複製 tsParticles 的碰撞物理。保留 100 顆上限、手機初始減量、離開視窗／隱藏分頁暫停、prefers-reduced-motion 初始與即時切換。尊重無動畫偏好，不為了可見度強制開啟。
- `tests/particles-browser.mjs` 接入 `npm run test:browser`；檢查實際 canvas 色彩／alpha、中英文、主題換色、滑鼠排斥、點擊 +4／上限、Logo 重播不誤觸、手機、離開視窗暫停與減少動態設定。新增可見度修正後，两組 CMS 的完整 verify 與粒子專項皆通過；兩主題截圖目視通過。隱藏分頁分支未另以真實背景分頁量測，不要擴大宣稱。

## PR #89 歷史確認狀態（後續版本見下節）

- PR #88 合併 `7d5e8393f35a605f4c6f0c8a2cb729a5902ae7a1`；PR #89 合併 `bddfc5c4e3b2a179c118ef8dd80383a7c89cc6de`。本次使用者明示略過外部審查、合併上線；不推廣為其他任務的永久偏好。
- 主站已原子切換至 `releases/20260909-bddfc5c`，保留前一版 `releases/20260909-7d5e839`，未重啟 Caddy。封裝 SHA-256：`362a7d75f302988c87ced71999e15d1156fabd5ef9420f35f12f1cad1f969bfe`。
- 合併後 CI 與 GitHub Pages 部署成功，主站／備援站各自跑 Brave 粒子專項全數通過；主站淺色正式画面亦已目視確認。相關 docs/architecture.md 已更新，本次乾淨 worktree 已清理，其他任務 worktree 保留。
- 以上為當時已驗證版本，不代表日後仍是最新；後續操作先查 Git main、兩站資產及 Inari current，避免覆盖其他任務更新。

## PR #91 淺色版微調與部署（2026-09-09 快照）

- PR #91 合併提交 `cd438873db47f159d6a9e17b25ba40923673f507`；已納入成員資料 PR #90，並保留其 `researchInterests` 驗證與測試。多任務同時更新 CMS 時，重定基底後應合併所有雙語欄位，不能覆蓋另一任務的驗證。
- 「2025 · October」不清楚的根因是共用 `.badge-muted` 設為 `border: none` 且白色底只有 0.03 alpha。淺色模式改為 1px `--line-strong`（`#a8b38b`）框、`#eef1e2` 實色底與 `#475533` 文字；同時涵蓋學期、年份與成員資訊標籤，保留深色 badge 配色。
- 使用者接受的微調方向：米白頁面（`#f7f8f2`）、灰綠細框、較小卡片標題、減少柔陰影與圓角；獲獎來源使用文字連結。保留 Logo、原始合照、首頁粒子與深色配色，不把「減少 AI 套版感」解讀成全面重做或移除既有互動。
- 手機選單線條使用不存在的 `--text-strong`，hover 使用不存在的 `--bg-soft`；改用既有 `--text`、`--surface-1`。光測選單能開關不足，還要驗證入口本身可见。淺色平面控制項保留明確 focus-visible outline。
- 網路搜尋未找到三場活動的日期／地點佐證，故移除泛用描述，不推測細節。`life.description` 改用既有 `optionalI18nText`，CMS 同步改選填；首頁與活動頁均條件渲染，填寫時仍要求中英文完整。不能只移除 Markdown 欄位而留下必填 schema 或直接索引。
- Inari 的 `command -v python3` 雖回傳 `/usr/bin/python3`，實際執行會出現 `No developer tools were found`，不能當成 Python 已可用。未安裝額外工具；改用系統 `shasum`、`tar`、`chown`、`chmod`、`ln` 與 BSD `mv -fh`。`mv -h` 可避免將暫存 symlink 移進 current 指向的目錄；須在同目錄執行 rename，切換前再次比對預期舊 symlink。
- 正式版 `current -> releases/20260909-cd43887`，前版 `releases/20260909-1f78f0b` 保留，未重啟 Caddy。此為當次實測，未來部署仍先讀取 current，不能硬套舊值。
- 封裝 SHA-256：`5553dd4c4dd20ec668270d2a336b1c75a8daa8e2698e5be5f667ccc8a14f20af`。兩站皆載入 `/_astro/awards.IwBPUfb2.css`，雜湊同為 `c56be4b7cc4f292025229091f0210a06ade59b85cd8c78fc376b29c59732d327`；檔名與雜湊是此次部署證據，不是永久設定。
- 最終 CMS disabled／enabled verify 各為 7 tests、0 failures，147 頁、2,776 links、708 images；完整 Brave 瀏覽器測試通過。合併後 CI／Pages 成功，兩站再驗證中英文、手機／桌面、深淺色、日期框、活動文案、HTTPS、canonical、靜態資產與 404。兩份相關文件已更新；本次分支、乾淨 worktree 及 preview 程序均清理。
- 使用者明確同意「同意 直接發佈部署」，授權 PR #91 略過此次無法使用的外部 bot 審查；沒有宣稱 bot 通過，不延伸為未來任務的永久豁免。當次組織安裝清單只有 GitRoll／Cursor，未列 Swear Review、Gemini、Codex；Google Developer Connect 未找到此倉庫連結。整合狀態會變，後續先重新查證。
