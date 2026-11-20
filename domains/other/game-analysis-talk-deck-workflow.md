---
title: 解謎讀書會遊戲分析簡報（Claude Slides artifact）做法
scope: domains/other
tool: Claude Artifact (Slides type), agy/Gemini, Pillow, pdftoppm
status: active
confidence: high
evidence: 2026-11 前程似錦與 TIS-100 兩場讀書會簡報實作；TIS-100 於 2026-11-17 實際報告完畢。
created: 2026-11-17
updated: 2026-11-21
tags:
  - slides
  - artifact
  - presentation
  - game-analysis
  - speaker-notes
---

# 遊戲分析簡報做法（遊戲設計研究社《解謎》讀書會）

## 內容

- 每場獨立：不跟前一場簡報對比；劇情不是核心就不講。講者署名 Swear。
- 段落：開場（基本資料、實機畫面）→ 機制 → 評分/玩家評價 → 類型定位與比較 → 帶得走的設計 → 3 個討論題 → 延伸推薦 → 謝謝＋資料來源。
- 投影片字少；完整口稿寫在 `<aside>` 講者備註（繁中）。自己的詮釋標「推論」。
- 類型起源這類問題要誠實寫：「是不是開創者」先查前作再下結論。
- 遊戲本體外的關鍵資料（如 TIS-100 的 PDF 手冊）要直接截原文放進投影片，並在備註點出「只有讀手冊才知道」。

## 素材

- 優先順序：使用者自己存檔的實機截圖（可開子 session 實際遊玩截圖）> 官方 Steam 截圖 > 玩家 YouTube 影片。不要自製假圖表。
- Steam 商店截圖可能來自舊版本：TIS-100 官方圖按鈕寫 PLAY，現行版本是 RUN。UI 文字一律以實機截圖核對。
- 圖片裁切用 `uv run --with pillow python`；macOS `sips -c` 的 cropOffset 不生效。純圖片 PDF 用 `pdftoppm` 轉頁再裁。
- 截圖若含排行榜好友 ID，報告前考慮裁掉。

## Slides artifact 機制

- 檔案：`project/deck.json`（order、sections）、`project/deck.css`、`project/slides/<id>.html`（一個 1920×1080 `<section>`，備註在 `<aside>`）。
- 更新：publish 帶 `url`、`root`、`file_path`=deck.json、`files` 只放改到的 slide；圖片先用 `asset: true` 上傳，引用回傳的 `/_blob/<id>`。
- 驗證：內建瀏覽器 viewport 1600×1000，導航後等約 18 秒；「This artifact didn't start」多等一下或按 Try again。逐張點縮圖看是否溢出，結束時 viewport 回 desktop。
- 版面溢出時先刪一列、縮 margin、縮短字，而不是縮字級。

## 審稿

- Gemini 審稿：`agy -p "<prompt>" --model gemini-4-hydrogen-high --print-timeout 600s`。每條意見都要查證；它會錯（按鈕名、彩蛋指令）也會抓到真問題。
- agy 非互動模式會拒絕 read_file，結果沒有輸出。要把所有內容直接寫進 prompt，開頭加「不要使用任何工具或讀檔」，並在空目錄執行（2026-11-21 GSAT 實習報告驗證過）。
- 要對講稿做深改時，先請 Gemini 列出缺少的資訊、問使用者，拿到答案再改寫，不要只用手邊素材硬寫。

## pptx 渲染檢查

- macOS 有 PowerPoint 時，用 osascript 讓它匯出 PDF（輸出在 `~/Library/Containers/com.microsoft.Powerpoint/Data/` 之下），再用 `pdftoppm` 和 `montage` 拼成總覽圖，比 LibreOffice 準。同名簡報如果已經開著，要先關掉。
- 白底證書改成深色底：在 Lab 色彩空間反轉 L 通道，再用 Screen 混合主題深色。照片一律用 sharp `fit:cover` 裁切，不要拉伸。
- 延伸推薦的發售年份、搶先體驗狀態要當天查證，並在交付時提醒報告前再確認。
