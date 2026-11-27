---
title: Majdata song-folder media and timing preparation
scope: projects/visual-maimai
project: Visual Maimai
machine: Windows
status: active
confidence: high
evidence: Verified against existing MaiCharts folders, MajdataEdit source, and a Baby Shark media preparation run on 2026-09-04
created: 2026-09-04
updated: 2026-09-04
tags:
  - visual-maimai
  - majdata
  - maidata
  - audio-video
  - bpm
---

- **開了音樂但譜面沒匯入、BPM/offset 變預設。** 歌曲同層放 `maidata.txt`、`track.mp3`（或 Visual Maimai 可開的 track.ogg）、`bg.png` 及可選無音訊 `pv.mp4`。Visual Maimai Open 要選 track，既有譜面用 File → Import maidata，儲存前讀 UI 值；新建空 `chart.json` 不能證明匯入成功。
- **offset 對到片頭瞬音，而非穩定拍點。** 從裁切音訊重新量 beat period 與 phase，`&first` 是秒數、起始 timing row 以它為準。每 `&inote_*` 起首顯式 `(BPM)`，仍保留 `&wholebpm`。Baby Shark 已驗 115 BPM、offset **0.22 s**，早期 0.10 s 鎖錯 transient，不沿用。正式原片保留。
- **空譜面 starter 被 parser 拒絕 `max() iterable argument is empty`。** 保留第一拍一個合法 placeholder `1` 和終點 `,,,E`；匯入成功後真正編譜時可移除。只準備媒體無須代造正式譜面。

[原始紀錄](https://github.com/swear01/agent-memory/blob/9b2d556e370f3a71736d5dc0a63f3b1e1c8c3a0e/projects/visual-maimai/baby-shark-media-prep.md)
