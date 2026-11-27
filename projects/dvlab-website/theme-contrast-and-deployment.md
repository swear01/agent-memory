---
title: DVLab 網站深淺色對比、粒子配色與主站部署驗證
scope: projects/dvlab-website
project: DVLab-NTU/dvlab-ntu.github.io
status: active
updated: 2026-10-02
---

## 兩站部署

- **Pages 已更新，Inari 仍是舊版。** 主站由 Inari Caddy 提供，GitHub Pages 是可直接開啟的備援。兩站都用 `PUBLIC_SITE_URL=https://dvlab.ee.ntu.edu.tw` 建 canonical、hreflang、OG 與 sitemap。部署 Inari 前比對同一 Pages artifact 的 SHA-256，新增獨立 release、核對原 `current` 再原子切換 symlink，保留上一版，不重啟 Caddy。兩站各自瀏覽驗證。若 `sudo -n true` 失敗，預檢和上傳都還沒完成上線，須由有權限的操作者切換。
- **preview 通過但瀏覽器回報 404。** 測試期間避免重建同一份 dist。以 `npm run verify` 分別驗 CMS disabled／enabled，用範例 OAuth URL 只能驗建置。瀏覽器核對深淺色、兩種語言、390／1280 px、normal／hover／keyboard focus、實際頁面與 canonical。

## 視覺與舊 PR

- **淺色徽章幾乎看不見。** 原 `--primary-2` 同時供深底文字與按鈕用。優先修共用元件的淺色文字，避免改全站 token；淡底文字以 4.5:1 為門檻。半透明背景與文字 opacity 要先合成，漸層和照片用實際像素確認。
- **沿用已部署的 CRA 設計時誤改版面。** 保留 Astro。沿用原始主持人 `ric.jpeg`、單一 PI 區及三組橫向成員軌（形式驗證、EDA/3DIC、Quantum）。透明 nav 高度由 ResizeObserver 更新 `--site-header-height`。首頁文化標語在 `#lab-introduction`；玻璃設計沿用 `--glass-bg*`，論文列表用較實的 panel，詳頁只留會議與年份的標籤 chip、同列 BibTeX 與複製鍵、共用 `SiteFooter.astro`。後續設計及字型要對照 repo 和 [PR #104 記錄](starfield-redesign-pr104.md)。
- **舊 PR 描述仍說暫緩或未部署。** #101、#102、#103 都有已合併與雙站部署的歷史證據。頁面有 typewriter markup 還要檢查對應 CSS 非空及 nowrap。#104 文件記錄 10/17 開 PR 時兩個 verify matrix 通過，尚未部署；診斷現況要讀當前 repo 與兩站。

[原始紀錄](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/projects/dvlab-website/theme-contrast-and-deployment.md) · [CRA parity #101](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/projects/dvlab-website/cra-visual-parity-pr101.md) · [標語 #102](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/projects/dvlab-website/typewriter-mottos-pr102.md) · [玻璃元件 #103](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/projects/dvlab-website/liquid-glass-chips-pr103.md)
