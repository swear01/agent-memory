---
title: Verto 設定畫面：圖示慣例與音量設定位置
scope: projects/verto
status: verified
updated: 2026-10-31
tags: [Verto, PHASE, PhaseKeys, PhaseAudio, PlayerPrefs, UI]
---

- 全螢幕圖示用四個向外的角括號，全螢幕中改顯示向內括號；「還原預設按鍵」用 undo 弧形箭頭（舊的 C 形箭頭使用者看不懂）。遊戲內不放說明文字，語意靠慣用圖示。
- 設定畫面是 `PhaseKeys`（`Assets/Phase/Runtime/PhaseMenu.cs`），標題與暫停選單共用；頁面 67x74，第 0-2 列按鍵、第 3/4 列 SFX/Music 滑桿、第 5 列 15x15 控制鈕。改版面要同步 `MenuTests`（Down 次數、Knob 座標）與 `Verification/KeysLiveInput.cs`。
- 音量 0..10 存在 PlayerPrefs `phase.volume.v1`（`"sfx,music"`），由 `PhaseAudio.SetLevel(bool sfx, int level)` 讀寫，預設滿音量；音樂音量乘在 `MusicBase` 上。
- 新增 Editor 測試要同步 `Verification/BuildWeb.py` 內寫死的各 filter 測試數。
- Unity CLI：`unity command <name> --project-path "$PWD" -- <args>`，`--project-path` 必須在 `--` 之前。
- 該 PR 的 Swear Review job 排隊停滯未完成，依 review-loop 規則在 CI 與 Gemini review 通過、僅剩 nit 時合併。
