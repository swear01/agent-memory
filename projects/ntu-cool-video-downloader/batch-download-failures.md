---
title: NTU COOL 18 支下載失敗：DASH 尾段、MP4Box 大檔與診斷保存
scope: projects/ntu-cool-video-downloader
status: active
updated: 2026-09-15
---

## 兩類失敗不能混為一談

一次 18 支失敗的批次有兩個根因；不要一律加重試或增加記憶體：

- 4 支：以 MPD 總長度向上取整估算視訊片段，多抓不存在的最後一段，HTTP 404。視訊與音訊的實際長度可差約 20ms；片段數應以各軌連續已解析 sample duration 和 init 宣告時間判斷完整性。
- 其餘 14 支：合併輸出超過 1 GiB，舊 MP4Box DataStream 倍增配置到 2 GiB，Brave 配置失敗。4 支尾段 404 中另外有 2 支也超過 1 GiB，因此大型輸出影響總共 16 支，兩類聯集 18 支。

舊程式已清掉個別錯誤紀錄，所以沒有還原原批次 18 份 stack trace。4 支有尾段 HTTP 回覆和實際片段證據；14 支大型輸出判定來自實際大小、舊配置路徑及相同 Brave 的 ArrayBuffer 配置重現。清楚區分原始錯誤紀錄與後續因果驗證。

## 最小且有證據的修正

- `utils/remuxer.js` 的 `mp4Blob` 以約 16 MiB 的序列化區塊組成 Blob，避免一次配置整個影片 buffer。MP4Box addSample 產生 per-sample moof/mdat；不要未經檢查就假設只有單一巨大 mdat，或誤認 DataStream buffer getter 沒有裁切有效內容。
- 先解析連續片段，僅在 sample 累計 duration 已覆蓋該軌 init 時間時省略估算尾段；不以 CTS 結束時間代替 sample duration。composition offset 可能掩蓋仍必要的短尾段，回歸測試要涵蓋。
- 保留必要尾段的下載與真正失敗，不以忽略全部 404 作為修復。大型 Blob 修正不代表記憶體無上限。
- 真實修復 MP4 為 1,258,977,190 bytes；ffprobe 視訊長 2940.480 秒、音訊 2940.501333 秒，ffmpeg 最後 5 秒解碼成功。此一支重現不代表全部 18 支重試成功。

## 錯誤可讀、可複製、可保存

- 反覆替換相同 textContent 會破壞使用者選取；改用狀態事件更新，保留 details 節點及沒變的文字。網址輸入框用 readonly 保留選取能力，不用 disabled。
- 保存逐支 error/errorDetails：stage、HTTP status、resource、track、segment、attempts 等；從批次建立與狀態轉換即保存最新報告，避免第一支未完成便重啟時退回舊批次。
- 複製、JSON 匯出使用固定快照；clipboard 被拒絕時顯示可手動選取的文字框。Stop 保留成功與失敗，retry 只重開失敗頁取得新來源，保留 lastError 與 retryCount；瀏覽器重開後不自動續傳。
- 診斷去掉 URL query、使用者資訊和 fragment，不能保存簽章或憑證。錯誤應傳到頁面，而非只顯示一般下載失敗。

## 已驗證版本

下載與診斷修正於 PR #15、1.2.2，合併 commit `afdfd11213f7e4ca3bf3bb22f01f54da7c742b46`。後續必須使用修復 service worker 啟動的 1.2.3；詳見同目錄 `service-worker-startup.md`。本次未將私有課程網址、原始媒體資料或簽章寫入共用記憶。
