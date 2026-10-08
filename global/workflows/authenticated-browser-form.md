---
title: Authenticated browser form continuity and submission gates
scope: global
status: active
created: 2026-09-03
updated: 2026-10-08
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
