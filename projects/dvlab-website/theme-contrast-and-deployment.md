---
title: DVLab 網站深淺色對比與主站部署驗證
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

## 最後確認狀態（需重新查核的快照）

- 2026-09-09 主站已觀察到 `current -> releases/20260909-5c9b733`；先前「等待管理員切換」狀態已過期。
- GitHub Pages 與主站各完成 137 個 URL 的深淺色掃描，無瀏覽錯誤，124 個詳細頁 badge 都是修正後的 8.75:1。主站 HTTPS、英文論文深層路由及不存在路由的 404 狀態亦確認。
- 本任務預備封裝（非實際啟用封裝的身分證明）SHA-256 為 `619dc7a762cd8c2dc212d6d65e5b7cf28eb0ffec5e59f4dd7ef2003ad108ddf2`；未保存任何密碼或憑證。
- 已清理本任務乾淨 worktree／分支並快轉本機 main；其他任務 worktree 保留。
