---
title: DVLab 成員入學屆別與照片對應
scope: projects/dvlab-website
status: active
updated: 2026-09-10
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
