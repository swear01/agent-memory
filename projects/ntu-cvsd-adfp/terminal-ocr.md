---
title: ADFP 終端畫面的本機 Vision OCR 與運算裝置邊界
scope: projects/ntu-cvsd-adfp
status: synthetic pipeline and bounded live ADFP Vision readback verified
updated: 2026-09-28
---

# ADFP 終端畫面的本機 Vision OCR

- ADFP 無法使用 SSH 時，經使用者授權，可用 loopback Chromium CDP 擷取 Guacamole 終端區域；截圖只在記憶體中傳給本機 OCR，不儲存影像。登入、密碼與 CAPTCHA 由使用者處理；只把任務必要、非機密的辨識文字交給 agent。
- 本機 shelf skill `adfp-vision-ocr` 用 `VNRecognizeTextRequest` 的 `.accurate`、`usesLanguageCorrection = false`。Apple Vision 公開的文字辨識等級只有 `fast` 和 `accurate`；`accurate` 是此 API 的最高精度檔位，並非所有終端截圖的全域最佳保證。Apple 文件指出，關閉語言校正會回傳原始辨識結果、改善效能，但可能降低一般文字準確率；命令與符號仍需實測。
- 一張 1400×420 合成終端圖、各五次暖機後測試：Tesseract 中位數 0.49 秒、Vision fast 0.16 秒、Vision accurate 0.22 秒；accurate 在這張圖精確匹配。這是單一合成畫面的測量，不代表所有遠端終端的速度或精度。
- 在 SwairM5 的 M5 MacBook Air 上，對相同設定的 `VNRecognizeTextRequest` 查詢 `supportedComputeStageDevices`：main stage 支援 Neural Engine、GPU、CPU；`computeDevice(for:)` 回傳 `nil`，表示沒有明確指定裝置，由 Vision 自選。支援 Neural Engine 不證明每次執行實際用了它；速度測試也不是硬體使用證據。M5 規格中的 GPU Neural Accelerators 與 16 核 Neural Engine 是不同項目。
- 實務上先維持 `.accurate` 與自動選擇運算裝置。節省 agent token 的主要手段是裁切終端、在本機辨識、只回傳必要行；是否強制 Neural Engine 應依相同畫面的準確率與耗時實測決定。精確路徑、數字、exit code 與錯誤文字需另做標記或讀回驗證。

## 2026-09-28 實際 ADFP 驗證與操作

- 真實 Guacamole 終端已用 Vision accurate 讀回 `ADFP_FINAL_PASS`、`RESULT ADFP_SUBMISSION_RECHECK_PASS` 與 `RESULT FILENAME_AND_CONTENT_EXACT_PASS`；截圖僅在記憶體中裁切、辨識並釋放，沒有保存影像。這證明本次標記讀回成功，不保證任意終端文字辨識正確。
- OCR 會把 `cvsd_hw1_v1.tar.gz` 的數字 `1` 看成小寫 `l`。精確檔名應在伺服器用字串比較驗證，必要時用 `printf "%s" "$name" | od -An -tx1`；ASCII `31` 才是數字 `1`。內容另以包內檔案 SHA-256 核對，勿只依畫面字形判定。
- 遠端互動 shell 是 tcsh。含 Bash 語法或 `>file 2>&1` 的批次命令，應包在 `bash -c '...'` 或上傳 Bash 腳本執行。
- 先檢查 VPN，再重用授權的專用 Chromium／loopback CDP 設定檔。Chromium 使用 macOS 真正全螢幕、隱藏上方工具列，遠端終端再以 F11 放大，可減少無關 UI；擷取仍只限任務必要區域。
- macOS `scutil --nc` 的系統服務名稱與 FortiClient 內的 profile 名稱可能不同。本機以 FortiClient profile 名稱查詢得到 `No service`，而 `scutil --nc list` 的實際 FortiClient 系統服務名為 `VPN`；使用已核對的服務識別碼，勿直接把 profile 名稱當成系統服務。
- `CVSD2026` 的 `tools/adfp-session.sh` 會檢查 VPN、核對專用 Chromium 的 process/profile，並等待 loopback 9223 就緒；`curl --noproxy '*'` 避免代理干擾，`ps -ww` 保留完整命令列。腳本不略過憑證警告；遇到安全警告需交給使用者手動確認。
- 啟動腳本七項檢查通過，也實測在設定錯誤代理時仍能重用目前連線。2026-09-28 已推送 `adfp-session` 分支，head `1631121bbfbb1e8545e097c55a6a6dc4070f971a`；PR #2 尚未合併。Swear Review 為 gate off，execution success 且有 4 findings，不能當作乾淨審查核可。下次使用前重新查驗 PR／分支狀態。

依據：Apple Developer 的 Vision `recognitionLevel`、`usesLanguageCorrection`、`supportedComputeStageDevices`、`computeDevice(for:)` 文件，以及 Apple MacBook Air 技術規格；本機 Swift API 查詢與合成畫面測試。
