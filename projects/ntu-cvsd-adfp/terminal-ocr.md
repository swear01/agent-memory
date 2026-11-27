---
title: ADFP 終端畫面的本機 Vision OCR 與運算裝置邊界
scope: projects/ntu-cvsd-adfp
status: synthetic pipeline and bounded live ADFP Vision readback verified
updated: 2026-10-20
---

- ADFP 先走 `adfp-vpn` 與既有專用 Chromium profile。SSH 無法用時，限使用者授權的 loopback CDP，裁切所需終端區域在記憶體內做 OCR，不保存截圖、不回傳認證內容。用 macOS 全螢幕、終端 F11 可降低雜訊。
- 本機 shelf `adfp-vision-ocr` 用 Vision `VNRecognizeTextRequest` 的 `.accurate`、`usesLanguageCorrection = false`。保持自動 compute device，`supportedComputeStageDevices` 只證明裝置可選，`computeDevice(for:) = nil` 不表示實際使用何種硬體。速度亦不能證明 Neural Engine 有參與。
- OCR 讀真實終端的 PASS marker 曾成功。數字 `1` 易被讀成小寫 `l`，精確檔名在 server 比較字串，必要時 `printf "%s" "$name" | od -An -tx1` 核數字 `1` 為 ASCII 31。包內容逐檔 hash，exit code 和命令診斷用明確 marker。
- 遠端互動 shell 是 tcsh。需 Bash 語法、`>file 2>&1` 時用 `bash -c '...'` 或上傳 Bash 腳本。舊 `tools/adfp-session.sh` 核 process/profile 用 `ps -ww`，loopback 9223 要用 `curl --noproxy '*'` 避免代理；現行認證與專用 profile 依 [[machines/swairM5/adfp-vpn]]。
