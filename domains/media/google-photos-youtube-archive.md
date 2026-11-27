---
title: "Google Photos 大型影片轉存範圍"
description: "Google Photos 大型影片轉存 YouTube 的門檻、批次與安全刪除規則。"
tags: [google-photos, youtube, storage, video]
category: domains
scope: global
status: active
updated: 2026-09-06
sources: [live Google Photos inventory, live YouTube Studio verification, local file verification, user decision]
confidence: high
---

# Google Photos 大型影片轉存 YouTube

- 此專案只處理 Google Photos 大小嚴格大於 100 MB 的影片，排除照片與較小影片。一般照片另有 [HEIC 工作流](google-photos-heic-full-scope.md)。上傳一律私人，按大小分批、本機只準備一批、跳過精確重複。未獲授權不公開、不永久刪除、不清垃圾桶。
- Studio 一次選檔上限（當時 15 支）不同於頻道動態 24 小時配額。私人、網頁和 API 都受頻道限額，hit limit 時保存來源和批次狀態。能持續處理時無需因 15 支固定等下一天。
- 上傳前同時核對 Studio 的影片、Shorts 與草稿。直式且不超過 3 分鐘的來源曾進 Shorts，僅查影片列表造成同一批重複上傳出草稿。先讀回真實項目身分、非零片長、私人狀態、處理完成，並在播放器實際確認進度；建立或草稿不能算可播放。
- 只有精確對應的來源、替代品私人且實際可播放，再按該批授權走可恢復垃圾桶。Drive File Provider 上 macOS `trash` 曾權限錯誤，改用已驗的 Drive trash API 或 Finder；本機 Downloads 可用系統垃圾桶。不要用 `rm` 取代。Google Photos、Drive 和本機的來源是分開核對的三個集合。
- 2026-09-06 提醒：既有私人 Shorts 有重複舊草稿，先對帳，不盲重傳或清理。當時有 14 支已驗播放但來源保留，其逐件回條與停留狀態見固定版本；數量是舊快照。排程已暫停，不自動復跑。

[整理前的完整證據與逐批紀錄](https://github.com/swear01/agent-memory/blob/06f50e3a42e850b37cffb6b6a4a446ef670e9625/domains/media/google-photos-youtube-archive.md)
