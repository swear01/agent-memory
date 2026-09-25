---
title: ADFP 終端畫面的本機 Vision OCR 與運算裝置邊界
scope: projects/ntu-cvsd-adfp
status: synthetic pipeline verified; live Vision ADFP terminal pending
updated: 2026-09-26
---

# ADFP 終端畫面的本機 Vision OCR

- ADFP 無法使用 SSH 時，經使用者授權，可用 loopback Chromium CDP 擷取 Guacamole 終端區域；截圖只在記憶體中傳給本機 OCR，不儲存影像。登入、密碼與 CAPTCHA 由使用者處理；只把任務必要、非機密的辨識文字交給 agent。
- 本機 shelf skill `adfp-vision-ocr` 用 `VNRecognizeTextRequest` 的 `.accurate`、`usesLanguageCorrection = false`。Apple Vision 公開的文字辨識等級只有 `fast` 和 `accurate`；`accurate` 是此 API 的最高精度檔位，並非所有終端截圖的全域最佳保證。Apple 文件指出，關閉語言校正會回傳原始辨識結果、改善效能，但可能降低一般文字準確率；命令與符號仍需實測。
- 一張 1400×420 合成終端圖、各五次暖機後測試：Tesseract 中位數 0.49 秒、Vision fast 0.16 秒、Vision accurate 0.22 秒；accurate 在這張圖精確匹配。合成 Chromium 截圖到 Vision accurate 到文字的流程已通過；真實 ADFP 終端只用 Tesseract 驗證過無機密測試標記，Vision accurate 在真實 ADFP 畫面的精度仍待驗證。
- 在 SwairM5 的 M5 MacBook Air 上，對相同設定的 `VNRecognizeTextRequest` 查詢 `supportedComputeStageDevices`：main stage 支援 Neural Engine、GPU、CPU；`computeDevice(for:)` 回傳 `nil`，表示沒有明確指定裝置，由 Vision 自選。支援 Neural Engine 不證明每次執行實際用了它；速度測試也不是硬體使用證據。M5 規格中的 GPU Neural Accelerators 與 16 核 Neural Engine 是不同項目。
- 實務上先維持 `.accurate` 與自動選擇運算裝置。節省 agent token 的主要手段是裁切終端、在本機辨識、只回傳必要行；是否強制 Neural Engine 應依相同畫面的準確率與耗時實測決定。精確路徑、數字、exit code 與錯誤文字需另做標記或讀回驗證。

依據：Apple Developer 的 Vision `recognitionLevel`、`usesLanguageCorrection`、`supportedComputeStageDevices`、`computeDevice(for:)` 文件，以及 Apple MacBook Air 技術規格；本機 Swift API 查詢與合成畫面測試。
