---
title: "libass 安裝與 ffmpeg 字幕濾鏡是不同層"
scope: tools/ffmpeg
status: active
updated: 2026-09-16
evidence_digest: b77ae33643e14e429a0f57ca4d06c8d4c976c9aff3a4a38cbef02cfaf1c85aa4
---

# libass 安裝與 ffmpeg 字幕濾鏡是不同層

歷史字幕處理先被歸因於 filter 語法，之後助手回報目前 ffmpeg 沒有 subtitles filter。使用者同意安裝 libass 後，助手又回報 library 已安裝，但既有 ffmpeg build 仍未包含該 filter，並提出更換 build。

檢查字幕能力時，確認實際使用的 ffmpeg binary 是否提供 subtitles filter，再區分缺 library 與建置時未啟用功能。安裝 library 不會自動改變既有 binary 的功能；替換後仍須以實際字幕輸出驗證。

來源只支持這段歷史診斷與更換 build 的計畫，沒有硬字幕完成的結果。套件名稱及 formula 功能會隨版本改變，不將當時建議當成目前通用安裝指令。
