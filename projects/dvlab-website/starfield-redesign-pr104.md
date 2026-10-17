---
title: DVLab 網站星空／星圖紙重新設計 PR #104
scope: projects/dvlab-website
project: DVLab-NTU/dvlab-ntu.github.io
status: active
updated: 2026-10-17
tags: [impeccable, liquid-glass, theme, typography]
---

## 狀態

- 2026-10-17 開 PR #104（branch `design/starfield-redesign`），CI `verify` 兩個 matrix 都 pass；**尚未 merge / 部署**。merge 到 `main` 才會部署 Pages，inari 仍需另外更新。

## 使用者確認的方向

- 深色：非常深的午夜星空（Milky Way、北斗七星／仙后座／獵戶座／天鵝座、閃爍星）＋ liquid glass。
- 淺色：使用者選「星圖紙」— 同一片星空印成星圖（極座標格線、星座連線、金色虛線黃道），玻璃是 vellum 半透明白。
- 金色保留為「星光色」，只用在 active nav、primary button、年份、tagline、motto。
- 設計紀錄在 repo 的 `PRODUCT.md` / `DESIGN.md`；星空素材由 `scripts/generate-sky.mjs`（固定 seed）產生 `public/images/sky/{night,chart}.svg` 與 `src/data/sky-twinkle.json`，不要手改 SVG。

## 架構重點

- CSS 收斂為 `tokens.css`、`base.css`（Jost @font-face + `.sky` 背景）、`components.css`、`pages.css`、`utilities.css`；舊 `cra-*.css`、`glass-ui.css`、`page-background.css`、`effects.css` 已刪。
- 所有按鈕統一 `.btn` / `.btn-primary` / `.btn-icon`（40px pill），chip 28px 不互動；外部連結加 `ExternalLinkIcon`。
- 主持人社群圖示改 `SocialIcon.astro` + `src/data/social-icons.mjs`（Font Awesome solid marks，LINE 只留對話泡泡並用 `fill-rule="evenodd"` 鏤空字）。

## 踩雷

- `scripts/verify-pages.mjs` 禁止 `<link rel="preload" as="font">`，加了會讓 173 頁全部失敗；字型只靠 `font-display: swap`。
- `npm run test:browser` 不在 CI 裡，需要 `PLAYWRIGHT_MODULE=<playwright>/index.mjs`；在 PR #104 之前 `main` 上就已經壞掉（還在測已移除的 member search），#104 已更新到可全過。
- 手機選單 sheet 放在有 `backdrop-filter` 的 `.nav` 裡面時，無法模糊整頁（見 `tools/browser/nested-backdrop-filter-sheet.md`），必須用不透明底色。
