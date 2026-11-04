---
title: Authenticated browser form continuity and submission gates
scope: global
status: active
created: 2026-09-03
updated: 2026-11-04
tags:
  - browser
  - forms
  - privacy
  - submission
---

# Browser selection

預設使用 Codex 內建瀏覽器，沿用該瀏覽器已授權的登入狀態。只有使用者明確指定或授權時，才操作私人 Brave 或其他外部瀏覽器；不可自行啟動另一個 Chrome 自動化 profile 作為替代。先確認控制工具實際連接的 browser identity。若連線失敗，回報控制限制，不要重啟、清除 profile 或搬移 cookies。此偏好取代早期的通用 Brave 預設，特定任務已授權的 Chromium/CDP 例外仍有效。

另一個自動化瀏覽器顯示登入頁，只能證明該 browser context 未登入，不能推論使用者日常 Brave、桌面 App 或 repository integration 已登出／停用。

# Form continuity

使用者已明確授權代填表單後，先完成所有可逆的欄位、選項與檢視步驟；不要每填一個欄位就重新詢問。使用者後續更正資料時，直接以新值取代舊值並繼續流程，不要重問已確定的欄位。

需要同意聲明或個資處理確認時，只在該安全邊界集中確認一次。圖形驗證碼、Email 驗證或 OTP 是人工交接點；讓使用者只處理必要的驗證，不要求其提供密碼或一次性驗證碼，完成後接續剩餘流程。使用者已明確授權該次正式送出時，完成檢查後接續送出，不重問；未獲授權或操作、目的地、資料、金額有實質改變時才要求確認。工具強制的人工交接或當下確認要求仍須遵守。

頁面卡住時，先等待、停止載入或重新檢查目前頁面，優先保留已填資料；不要因一次控制元件失敗就反覆停住或清除整份表單。若必須改由本機瀏覽器接手，交接內容要包含網址、已完成／未完成步驟、欄位值、正文與最後應停下的不可逆操作。

# Evidence and privacy boundary

填完欄位、進入檢視頁或收到 `started` 類回應，都不等於正式送出。完成後要看到官方受理畫面、案件編號／查詢碼或可對應的確認通知，才能宣稱送出成功。

不要把姓名、電話、地址、Email、帳號、驗證碼、Cookie、登入狀態或私人表單正文寫入 shared memory；只保存可重用的流程規則與去識別化的失敗教訓。

# macOS CUA 控制逾時與 PDF 預覽

2026-10-01 的實測中，CUA extension 分頁控制在 `Emulation.setFocusEmulationEnabled` 逾時時，原生 Brave 的 accessibility 操作仍可用。這是可嘗試的不同控制面，不代表必須重開瀏覽器或清除 profile。當 bundle ID 因 Sparkle 更新快取副本而模糊時，可用已核實的主應用程式完整路徑定位。

AX 點擊後第一次觀察沒有變化，可能是載入尚未完成；先重新觀察頁面、網址與可見內容，不盲目重複點擊。原生座標操作若回報 `noWindowsAvailable`，不代表 accessibility 操作必然無法使用，兩者應分開判斷。不得因 native 操作成功就宣稱 extension 的 CDP 問題已修復。

文件按鈕若開啟 PDF 預覽，可透過瀏覽器 Command+S／下載儲存。儲存視窗初期按鈕可能 disabled；確認實際檔名欄與狀態後再儲存。完成後驗證本機檔案格式與內容，而不是把控制 API 回覆或下載按鈕點擊當成交付證據。

# Google Forms 編輯與回覆驗證

2026-11-04 實測：Drive connector 搜尋只回傳連結的回覆試算表，不能據此認定原始表單不存在。內建瀏覽器的 Google 表單首頁搜尋可找到原始表單；確認結果標示「Google 表單」，不要把「回覆」試算表當作表單編輯頁。截圖轉錄的不透明 ID 容易混淆字母，直接用網站搜尋結果取得正確網址，不輪流猜網址。

同一工作階段的 Brave extension 可讀頁面但點擊反覆逾時，使用者指定內建瀏覽器後，內建瀏覽器可正常完成相同任務。各控制面分別驗證；舊的 `Local browser use is disabled in Work settings` 不代表之後仍停用，也不能把讀取成功當作點擊可用。

編輯選項後先確認「所有變更都已儲存到雲端硬碟」，再重新載入核對題名、選項與「其他」是否已移除。既有回覆仍可能保留舊選項字串，改選項不等於改歷史記帳資料。查今天回覆時區分提交時間與消費日期，並交叉核對表單回覆與連結試算表；不為驗證而新增測試消費。私人表單網址及消費明細不存共享記憶。

# Google 相簿取檔與 Google Forms 檔案上傳

2026-11-04 實測（Claude 桌面 App 內建瀏覽器，已登入 Google）：

- Google 相簿剛上傳的文件照片，中文關鍵字搜尋可能尚未索引；以照片上的英文字（公司名等）搜尋或「最近新增」（`/search/_tra_`）較快找到。開啟單張照片後按 `Shift+D` 會下載原檔到本機 `~/Downloads`，檔名為數字時間戳 `.jpg`；以 `ls -t` 與 `sips` 核對新檔後再使用。
- Google Forms 的「新增檔案」是 Drive picker iframe（同源，可讀 DOM），但無法從頁面 JS 塞入本機檔案：`fetch('http://127.0.0.1:…')` 直接 `Failed to fetch`（伺服器未收到請求）；`window.open` 在內建瀏覽器會取代目前分頁而非開 popup，表單頁會被導走。Forms 草稿會自動還原文字欄，但單選題可能需重選並以 `aria-checked` 核對。
- 可行做法：把檔案複製到 Google Drive for desktop 的同步資料夾（`<home>/Library/CloudStorage/GoogleDrive-<account>/我的雲端硬碟/`），幾秒後在 picker 的「我的雲端硬碟」分頁搜尋檔名、雙擊插入即可。表單提交時會複製一份，事後可刪掉 Drive 暫存副本。
- 只收 PDF 的上傳欄位，多張圖片可用 `uv run --with pillow` 以 `Image.save(..., save_all=True, append_images=[...])` 合併成單一 PDF（系統 Python 未裝 Pillow）。
