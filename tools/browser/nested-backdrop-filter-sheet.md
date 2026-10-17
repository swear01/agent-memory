---
title: 巢狀 backdrop-filter 的下拉選單會透出底下文字
scope: tools/browser
tool: chromium
status: active
updated: 2026-10-17
tags: [css, backdrop-filter, liquid-glass, dropdown]
---

## 現象

- 父元素（例如浮動 glass header `.nav`）有 `backdrop-filter`，子元素是 `position: absolute` 溢出父框的下拉 sheet。
- 子元素自己的 `backdrop-filter` 只會取樣到父層這個 backdrop root，不會模糊頁面內容。
- 在 Chromium（Playwright headless）實測：sheet 底色用 `rgba(..., 0.97)` 甚至 `0.98`，底下頁面的大標題仍然肉眼可見，比 3% 透明度預期的明顯得多。

## 解法

- sheet 改用**完全不透明**的顏色（例如深色 `#0a1022`、淺色 `#f8f5ed`），保留 sheen gradient 和 inset rim 仍有玻璃感。
- 驗證：Playwright `elementFromPoint` 確認 sheet 在最上層後，用 `deviceScaleFactor: 2` 截圖比對；加 `.nav{backdrop-filter:none}` 或 sheet 不透明都能消除透字，確認是巢狀 backdrop 的合成問題。
