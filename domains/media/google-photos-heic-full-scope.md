---
title: Google Photos 全量 HEIC 範圍與 API 批次回驗流程
scope: domains/media
status: active
updated: 2026-09-15
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
- 瀏覽器主要處理 Takeout 範圍／匯出及必要登入；原件身分、相簿與可恢復垃圾桶可沿用下節已實測的網頁 API 批次路徑。官方 Library API 沒有原件二進位覆寫或照片刪除功能。
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


## 精確批次清理：2026-09-15 實測與更正

- 使用者要一次連續完成已授權清單，不能每 50／100 張就停下等待。官方上傳每次請求上限、網頁刪除 payload 批量與任務總量是三件事。刪除採每次 250 張、檢查點自動續做；以腳本保存完整回條，只回報摘要，避免逐張 UI 操作與反覆輸出大型 JSON 消耗 token。
- 可沿用經檢視的 Google Photos Toolkit／google_photos_web_client 網頁 API：GetItemInfoExt 的實際回應建立 mediaKey → dedupKey，MoveToTrash 使用 `XwAOJf` 的 dedupKey 陣列。這是未公開接口，250 為實測批量而非 Google 保證上限；須依当前工具、授權及協定重新核對可用性。
- Takeout 的 HEIC 網址 ID 與主圖庫 canonical mediaKey 可以不同。本次 21,173 張皆透過實際 dedupKey 對應成功；不能直接拿官方 API ID、照片 URL、檔名或 SHA 當成可互換的刪除 ID，也不能因 URL 不同就判定 HEIC 不存在。
- Python client 原始 parser 每個 JSON 行只取第一個 `wrb.fr`；實際 250 個請求、250 個 frame 曾被錯解析成 225 筆。須展開同一行所有 frame，再核對回應數、request ID 集合與逐筆項目 ID，不依回應順序配對。
- 本次使用內建瀏覽器受支援的目前來源 CDP 能力更新既有登入，僅供本機 Photos API 使用；暫存登入檔限制權限、限定 Google Photos 來源、設定 HTTP 逾時與停止未預期重新導向。任務結束移除暫存登入檔，絕不寫入記憶或 Git。這不代表其他環境自動具有相同能力或可繞過安全挑戰。
- 即使逐次保存 Google 更新的 cookies，本次 API 工作階段仍曾失效；瀏覽器登入當時仍有效，更新目前工作階段後可從回條續做。不把缺少登入頁資料、空 RPC 回應直接當成照片遺失、刪除成功或 bot 封鎖；沒有驗證固定失效時限或並行造成失效的根因。
- 刪除前確認來源／HEIC 逐張證據與清單不相交、HEIC 仍存在，保留來源的自訂相簿、最愛與封存狀態。本次 4 張先保留封存並回讀。每批先落盤刪除意圖，再送 API；結果不明先查垃圾桶，不盲目重送。回讀精確 mediaKey／dedupKey 與對應 HEIC，收尾核對垃圾桶前後差集精確等於清單。
- 日期不可只用完全相等的 epoch 毫秒作為硬門檻，也不可把所有失敗概括成時區錯誤。先計算每張的實際差值、時區與既有容差。本次兩張 pilot 分別差 8 小時與 0.481 秒；先前報告錯說兩張均差 8 小時，已更正。0.481 秒是使用者允許的微小差異，不能僅因此擋下；8 小時仍待釐清，不能因牆鐘畫面一致就宣稱絕對時間相同。ICC 原始位元組或缺省狀態仍嚴格保留，不套用數值容差。
- Drive 暫存清理不能只掃 `Takeout`。本次另漏了 `Google Photos HEIC staging 2026-09`，內含兩卷舊 ZIP、進度文件與索引，共 6 檔、4,304,438,890 bytes。先核對本機 ZIP 大小／MD5並另存小檔快照，再把精確檔案與空資料夾移入垃圾桶並回讀；不能把 FileProvider 路徑顯示當作完整獨立本機備份。
- `rclone lsjson <remote:folder> --stat` 在此得到虛擬根目錄、空 Name 且無 ID；不要據此聲稱資料夾 ID 已驗。從父資料夾列舉並對照已知 ID，另確認檔案 ID、大小／MD5。雲端清理不等於永久刪除或配額已釋放；不清空垃圾桶，不重新啟用停用的定時任務。

### 已核對收尾快照（後續操作前仍須回讀）

本輪新增清理 21,063 張原件，先前 360 張重新確認，已核對累計 21,423 張；不是全庫每張都完成轉檔。當時仍保留 3 張：一張雲端 1393×972 與備份 1958×972 不同、一張時間差 8 小時、一張僅差 0.481 秒且不應再以此單一差異阻擋後續清理。這次記憶更正沒有新增雲端刪除。185 個 Takeout 匯出檔以及額外 staging 資料夾 6 檔均已回讀在可還原垃圾桶；HEIC 與本機備份保留。

可復用程式：`<project-root>/work/photos_web_cleanup.py`、`photos_bulk_trash.py`、`photos_pilot_trash.py`。後者日期完全相等檢查尚未改成已接受的微小差異規則，續做前先修正與驗證，不能把此筆記當成程式已修好。具體身分与回條僅留本機：`<project-root>/work/photos-web-cleanup-20260915/cleanup-goal-result.json`、`pilot-held.json`、`trash-batches/`、`<project-root>/work/heic-staging-cleanup-result-20260915.json`；讀取原始數值而非泛化文字原因。
