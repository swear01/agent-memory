---
title: TOEFL 2026 external practice downloads and browser limits
scope: projects/toefl-practice
status: verified-with-limits
updated: 2026-10-21
---

# 新制托福題目下載

以下路徑均相對私人托福衝刺 `<project-root>`，不將題目原件、帳號資料、本人作答或錄音提交共享記憶。

## 已保存內容

- `參考資料/學而思考滿分_20261021/題目合輯.md`：透過本人已登入的 Brave 保存組句 20、Email 3、學術討論 3，共 26 道不重複寫作題；組句 JSON 保留空格數與固定詞位置。
- 同資料夾的口說僅是部分素材：4 個 MP3、3 張圖片及指示，不是完整 7 句複誦／4 題訪談題組。MP3 完整解碼通過，圖片尺寸可解析。
- `參考資料/網路新制模擬題_20261021/使用入口.md`：ETS 官方 Preparing Students 頁列出的學生版 1–2、教師資源版 1–5，共 7 份 PDF 與 7 個對應音檔 ZIP；原 ZIP 保留並已解壓。
- 7 份 PDF 均可解析且檢出 Build a Sentence、Write an Email、Listen and Repeat、Take an Interview；7 個 ZIP CRC 通過。學生版 1／2 與教師版 2 曾用作本地教材參考，未確認為本人未見題。
- 同資料夾 `TST-Prep/音檔目錄.md`：TST Prep 公開 7 句複誦、4 題訪談、兩段口說指示及 5 段聽力樣例，共 18 個 MP3。不是其完整四科模考下載。
- 官方解壓媒體與 TST Prep 合計 110 個媒體檔通過 FFmpeg 完整解碼；32 個原始下載檔的來源、大小、SHA-256 與核對結果保存於 `下載清單.json`。
- `00_使用導覽.md` 已新增模擬題下載入口；既有課表、作答與完成進度未改動。

## 瀏覽器與驗證界線

- 此次 Codex 內建瀏覽器只有開啟網址工具，沒有可用的讀取／點擊介面；使用者後來接受 Brave。Brave 的 AppleScript `execute ... javascript` 可讀 DOM、點擊題型與 Next，並擷取已載入的媒體網址；不能把開啟頁籤成功當作看見題目。
- 考滿分進入練習會新增平台練習紀錄。沒有填答案或手動交卷，不計為本人作答完成。
- 口說頁起初要求麥克風授權，後續偵測權限已 granted 並進入回答倒數；已退出口說流程。未查核平台是否保留那段錄音；後續只為取題時，須留意自動錄音流程。
- Magoosh 新制模考入口要求 Email；ETS 線上完整樣題入口要求 Email 與地區。只讀取入口，未送出表單、註冊或完成模考。
- TOEFLMockTests 的第一回頁面標示免費、免註冊，後續回數列 Premium；僅確認入口與設備檢查內容，未實測完整流程或評分。
- 檔案解碼、PDF 解析與 QMD 索引均不等於逐題人工核聽、難度等值、第三方估分準確、雲端同步或本人練習完成。
