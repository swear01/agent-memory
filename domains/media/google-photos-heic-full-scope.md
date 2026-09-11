---
title: Google Photos 全量 HEIC 範圍與 API 批次回驗流程
scope: domains/media
status: active
updated: 2026-09-11
tags: [Google Photos, Takeout, HEIC, Drive API, rclone, SHA256, Motion Photo, in-app-browser]
---

# Google Photos 全量 HEIC 範圍與 API 批次回驗流程

- 使用者要求全部照片轉 HEIC 與清理時，初始來源備份的 Takeout 必須選 Google Photos 的全部年份、全部相簿。不得因磁碟空間不足或想先試做，就自行只匯出某一年；抽樣驗證不能取代完整工作範圍。
- 空間限制只影響下載、解壓、轉檔及驗證的批次大小。全量匯出可使用 2 GB ZIP 分卷，後續逐卷處理並追蹤全部進度。
- 使用者明確要求使用 Codex 內建瀏覽器，不占用其滑鼠。此工作使用 in-app browser 的頁面操作；不得自行切回原生桌面控制或 Brave 前景操作。
- Takeout 相簿控制項可能延遲載入。初始全庫備份提交前等待並確認「已包含所有相簿」，必要時開內容選項核對所有年度均勾選；提交後確認「Google 正在建立」的成功狀態。
- 2026-09-06 的錯誤是只選 2024 年。更正後已於當日 21:53（Asia/Taipei）在內建瀏覽器建立全部相簿匯出。這只證明匯出已建立，不代表下載、HEIC 轉換、回傳或清理已完成；後續須重新確認即時狀態。
- Android Motion Photo 規格支援 HEIC，但普通靜態圖轉碼會遺失動態部分；須保留影片、聲音與 XMP 並正確封裝。Apple Live Photo 是照片與配對影片。不能把動態照片一律判定不能轉，也不能未經 Google Photos 回傳播放驗證就刪除原件。
- 靜態照片既定試轉品質為 80（有損），保留拍攝時間、位置等資料。原件與 Takeout JSON 保留到回傳驗證完成；清理必須精確對應且確認占用容量，不能依年份推定免費與否；不自動清空垃圾桶。
- 大型影片轉存工作中的「照片不處理」只適用該影片流程，不得套用來否定後來已授權的全量照片 HEIC 工作。

## 優先流程：API 上傳，Takeout 到 Drive 批次回驗

- 使用者明確要求降低 token 與瀏覽器操作成本。預設沿用已驗證的 API／腳本批次流程；先讀既有程式、狀態與回執，避免重造工具、重複上傳、逐張讀取整頁。子代理只接有界且獨立的查證或清單工作。
- 以 Google Photos Library API 上傳原始畫質至每批專用驗證相簿，持久化來源 SHA256、上傳 SHA256、API mediaItem ID、各來源 URL 與原相簿關係。API 先核對回傳身分、數量、尺寸、拍攝時間；未明確成功的建立請求先對帳，不能盲目重試。
- 多個 API 批次上傳與回讀完成後，在官方 Takeout 網頁精確合併選取這些待回驗的驗證相簿，設定單次 ZIP 匯出並「新增至雲端硬碟」。這是新檔的批次回驗，不是縮小初始全庫來源範圍。其他年份與相簿仍須完整處理；不要重新匯出全庫來驗證一小批。
- 等匯出完成，再沿用既有授權的 rclone／Drive API 下載該批 ZIP。先核對 Drive 檔案大小與 MD5，再驗 ZIP CRC、媒體清單與每個 HEIC 的 SHA256；不能把 Drive 上傳副本直接拿來替代 Photos 回驗。ZIP 必須由 Google Photos 的 Takeout 產生。
- 2026-09-10 實測：API `baseUrl=d` 回傳 PNG（1,282,574 bytes），但 Takeout → Drive API 取得 HEIC（254,414 bytes），與上傳檔 SHA256 完全一致。證據位於 `<project-root>/work/cloud-pilot-takeout-20260910/verified.json`、`download-receipt.json` 與 `<project-root>/outputs/google-photos-download-verification.md`。僅證明這一張；其餘每張仍須回驗，不能宣稱全部已替換。
- Library API 與 Picker 的 `baseUrl=d` 未承諾原始容器或逐位元組相同，且排除位置 EXIF。API 回應變 PNG 不代表 Photos 儲存原件被轉掉。只有下載檔 SHA256 與上傳檔相同，才可證明容器、影像與全部內嵌中繼資料未改；回應的 MIME、原始畫質標示、尺寸或下載完成提示都不足。
- 內建瀏覽器 `Invalid InterceptionId` 在本次下載事件工具重現；根因未確認。不要盲目反覆重試、任意改安全設定、接管使用者滑鼠，或聲稱升級 Playwright 一定修復。優先用上述 Takeout → Drive API 的已驗證路徑。
- 瀏覽器只處理公開 API 不支援的必要步驟，例如 Takeout 範圍／匯出、歷史原相簿、原件身分與可恢復垃圾桶操作。Library API 沒有原件二進位覆寫或照片刪除功能。
- API productUrl、網頁與 Takeout 的 URL 表示可能不同；本次以同一個只有一張照片的專用相簿、完整檔案雜湊、日期與可見資訊核對。保存別名與證據，不能泛化成只憑 SHA 合併多個原照片身分；同名也不證明同一張。
- SHA 相同的回下載也不能單獨授權刪原件：原始 ICC 位元組或缺省狀態、實際色彩、動態部分、重要中繼資料須先在轉檔端通過；歷史身分、日期、原相簿、共享狀態與容量計費仍須核對。完整通過才將精確舊項目移入可恢復垃圾桶，不清空。
- Takeout 有快照延遲與非固定處理時間，漏項不得算通過；Drive ZIP 會占配額，按批次與可用空間安排。預設不恢復已被使用者關閉的定時任務。
- 後續上傳以原檔名主幹加 `.HEIC` 作為 API fileName，別把工作目錄的 `converted.HEIC` 當正式名稱。既有測試檔不能為改名字而盲目重傳。

官方查證入口：Google Photos Library 的 access-media-items / upload-media、Picker 的 media-items、Google 帳戶說明 3024190（Takeout / Add to Drive）、Drive API manage-downloads（`files.get` + `alt=media`）。需要新行為時再查最新官方文件，不把一次實測推成無條件保證。


## 減少 Takeout 密碼金鑰驗證（2026-09-11 官方查證）

- Takeout 匯出／下載是敏感操作，Google 可要求近期重新驗證身分；登入狀態不等於已通過敏感操作驗證。官方 Takeout FAQ 說明未近期驗證時會再次要求密碼，開啟兩步驟驗證時可能多一步。密碼金鑰是可用驗證方式之一；沒有查到可保證永久免驗證的官方開關或固定免驗證時長，不能承諾驗證一次永遠有效。
- 不想使用密碼金鑰時，可在當次挑戰選「試試其他方法」，只使用 Google 實際提供的方式。Google 帳戶「安全性與登入」中的「盡可能略過密碼」（Skip password when possible）可關閉密碼金鑰優先登入，改為先要求密碼；這不等於取消 Takeout 敏感操作驗證或兩步驟驗證。不要替使用者擅改安全設定、刪除金鑰或停用兩步驟驗證。
- 根本流程修正：API 傳輸批次、檔案驗證批次與 Takeout 匯出次數分開安排。API 可以每次最多 50 張建立並逐批回讀，但不應每 250 張就建立一個 Takeout。只要身分、檔案與容量清單完整，先完成較多已授權候選，再將多個專用相簿合併進一次匯出；合併上限依暫存容量、payload bytes 與可恢復性決定，不能依單一年份或武斷張數縮小全量目標。
- 匯出請求必須在納入的各批上傳及 API 回讀完成後建立。保留每批 source SHA256、output SHA256、mediaItem ID 與確切相簿映射；回验時按實際相簿加雜湊核對完整聯集，不能讓同一已知雜湊被放在錯誤相簿也通過。已建立的匯出不假定會納入後來的上傳；新增相片可能有快照延遲。
- 匯出目的地維持同帳號 Drive，完成後沿用已授權的 rclone／Drive API 自動下載、MD5／CRC／SHA256 與 JSON 回驗；因此不必逐卷經 Takeout 網頁下載挑戰。這只能減少互動，不保證建立匯出免驗證，也不能用直接上傳到 Drive 的副本冒充 Photos 回下載。原件在完整回驗及原件資訊核對前保留。
- 維持使用者指定的內建瀏覽器登入狀態；完成身分驗證返回管理頁時，先查既有作業是否已建立／完成，勿因 URL 改變就重按建立。不要重複刷新登入流程、搬運驗證參數、保存私密回呼或嘗試繞過挑戰。
- Data Portability API 確有 30／180 天授權及同一資源每 24 小時再次匯出的機制，但截至本次查證，完整官方 scopes 沒有 Google Photos；maps.photos_videos 是 Google Maps 投稿，不是相簿。官方地區清單為 EU 成員國、瑞士與英國，不含台灣；也不能只依時區推定帳號國家。此 API 目前不是本次 Google Photos HEIC 回驗替代路徑；未確認產品與帳號資格前，不建立新 OAuth／計費專案或承諾可用。
- Google Takeout 提供定期匯出與 Photos 新增／更新資料匯出，但週期不適合這次即時遷移；本次頁面曾有月／雙月選項，官方說明列雙月。不可將此當成每日即時 API，亦不可自行重啟使用者已關閉的定時任務。

官方來源：
- [Takeout 與重新驗證 FAQ](https://support.google.com/accounts/answer/3024190?hl=en)
- [敏感操作的身分驗證](https://support.google.com/accounts/answer/7162782?hl=en)
- [密碼金鑰與替代登入方式](https://support.google.com/accounts/answer/13548313?hl=en)
- [Data Portability API 授權時效](https://developers.google.com/data-portability/user-guide/time-based)
- [完整支援 scopes](https://developers.google.com/data-portability/user-guide/scopes)
- [支援地區與帳號限制](https://support.google.com/accounts/answer/14452558)
