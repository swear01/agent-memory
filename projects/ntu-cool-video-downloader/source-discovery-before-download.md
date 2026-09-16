---
title: "播放器已載入不代表擴充功能已找到影片來源"
scope: projects/ntu-cool-video-downloader
status: active
updated: 2026-09-16
evidence_digest: 275380e10e99f5d523602aa1dd9fcb382bb251448a006fd3b204524ff0b24dae
---

# 播放器已載入不代表擴充功能已找到影片來源

歷史助手自述在新版播放器頁面，popup 與 Download 都顯示 No video URL found，且沒有 MP4 或暫存下載檔。

先驗證請求捕捉、iframe 與來源狀態，再追下載或 remux。沒有 URL 可縮小失敗階段，但單憑畫面不能證明特定 webRequest 路徑就是根因。來源沒有修復結果，不能把其他工具能捕捉請求當成此擴充功能已支援。
