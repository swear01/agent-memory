---
title: DVLab 成員資料、研究標籤與簡潔文案
scope: projects/dvlab-website
status: active
updated: 2026-09-11
---

## 入學屆別與狀態

- 表單 Admission cohort 指入學屆別，填民國年，允許本人自認的同屆分組；網站 `cohort` 用 R 短碼。R13 = 113 學年度（2024），R15 = 115 學年度（2026）。不要當作畢業年、上傳年，也不要僅憑學號覆蓋本人資料。
- PR #94 將中英文成員列表與個人頁明確標為入學年度；陳亭瑋（spongebobaa16）與簡郁（r13921049）依使用者確認改成 alumni，cohort 仍為 13。畢業狀態與入學屆別分開維護。
- 2026-09-10 已逐筆檢視 36 人：13 份本人表單、13 份本人 JSON 屆別資料夾；10 人沿用舊網站，其中一人另有本人就讀期間佐證。其餘 9 人仍缺新的獨立入學年佐證，保留原值，不可宣稱全員均獨立確認。

## 照片與資料同步

- 使用者明確確認 R15 的 `IMG_2457.JPG` 是詹維宏（Eric，chan-wei-hung）；PR #95 已將原圖設為 `public/member/images/chan-wei-hung.jpg`，Markdown avatar 指向 `/member/images/chan-wei-hung.jpg`。此確認取代先前的照片待確認狀態。
- 詹維宏、賴正翰、陳威霖均為 R15；截至上述日期，後兩人的照片仍未找到可靠對應。不得依外貌或僅憑檔案擁有人猜測身分。
- PR #93 已補上晚到表單與既有 JSON 的缺漏；後續同步仍重讀來源並依穩定 member id 合併，保留原始表單、JSON、照片及檔名大小寫。不要將 36 人等本次數量當作永久清單。

## 已驗證發布與再驗證邊界

- PR #95 合併提交 `9803548cabadc9625da57e2d7dc0a764cf781d7a`。兩組 CMS verify 與該 head 的 Cursor Bugbot 審查通過，主站及 Pages 的中英文名單／個人頁、手機／桌面照片均載入成功；兩站原圖 SHA-256 與使用者確認的原圖一致。
- 此次 Inari release 為 `releases/20260910-eric-91b5c65`；只是當次快照，未來部署前重讀 current，保留前版並原子切換。兩站需分別驗證，Pages 成功不等於 Inari 已更新。
- Cursor Bugbot 本次可由 `bugbot run` 手動觸發並回覆 clean review；先前「外部審查不可用」不是永久狀態。未來須確認審查對應最新 head，不能沿用舊版通過證據。

## 公開頁面文案保持簡潔

- 使用者明確要求網站不出現解釋性的維護文案。PR #96 移除成員頁中英文的「屆別採入學學年度……不是畢業年份」段落；入學年度定義仍保留在內容指南與記憶，頁面只保留年度標籤。不要因資料校正而把內部規則重新加到公開頁面。
- PR #97 將中文課程按鈕「課程目錄（NTU）」簡化成「課程目錄」，連結不變。此偏好適用於頁面標籤：避免不必要的括號與補充說明。

## 研究主題是標籤，自我介紹保留敘述

- 使用者接受並已完成 PR #98：`researchInterests` 改為可省略的 `{ zh, en }` 物件陣列，每筆都是短標籤，使用 `z.array(i18nText).optional()`；不能再寫成舊的雙語長字串。CMS 同步使用 list widget，每筆中英文必填，空清單或未提供均可。
- 個人頁在姓名區下方顯示可換行的標籤，移除獨立的研究興趣大卡片。沒有研究標籤時才顯示原本的廣義 area badge；成員列表仍保留既有 area 分類。
- 逐人根據本人提交內容整理與統一同義名稱，不要用逗號自動切割或推測缺漏。研究方法、完整句子與個人嗜好放在 `bio`，保留原本內容；文學、繪畫、吉他、排球不作為此實驗室的研究主題標籤。
- 本輪實際遷移 21 位有研究興趣資料的成員；早先口頭初估 22 位已更正。36 位成員的原始簡介、身分、屆別、狀態、連結與照片均已核對保留。數量是本輪快照，後續需重新統計。
- 維護入口：`docs/content-guide.md`、`src/utils/content-schemas.mjs`、`src/components/MemberPage.astro`、CMS 設定與 preSave 驗證。`tests/research-tags-browser.mjs` 已接入瀏覽器測試，涵蓋中英文、深淺色、手機／桌面、長標籤與未提供主題。
- PR #98 合併提交 `08befeec2eccbebf9bb8e40d10680a9e3bb442a6`；兩組 CMS verify、當次 head 的 Cursor clean review 通過，兩站各 72 個成員語系頁與已驗證建置逐位元比對一致，線上標籤瀏覽器檢查通過。Inari 當次 release 是 `releases/20260911-tags-e560f61`；未來操作仍重讀 current，不將此快照當作永久最新版本。
