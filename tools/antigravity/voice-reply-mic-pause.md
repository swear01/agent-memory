---
title: Antigravity voice-reply 麥克風偵測與打斷行為
scope: tools/antigravity
tool: voice-reply CoreAudio hooks
status: active
confidence: high
evidence: >-
  2026-11-26 在 macOS 實測 CoreAudio `prs#`/`piri`/`ppid` 可精確區分 Antigravity
  進程是否正在錄音；移除 `hooks.json` 的 `PreInvocation` 後，舊語音只會在下一輪
  `Stop`（`--speak`）開播時才被停掉。
created: 2026-11-26
updated: 2026-11-26
tags:
  - antigravity
  - voice-reply
  - coreaudio
  - macos
---

# Antigravity voice-reply 麥克風偵測與打斷行為

`voice-reply` 插件位於 `~/Documents/transfer_MAC/stow/agents/.gemini/config/plugins/voice-reply`（symlink 到 `~/.gemini/config/plugins/voice-reply`）。

## 麥克風競態與跨 App 優先權

- **切忌在播放入口用 `ignore_always_on_mic = is_mic_in_use()`**：若使用者在 TTS 合成期間或 chunk 切換瞬間已按下語音鍵，會誤判成常駐麥克風而整段禁用 `SIGSTOP` 暫停，直接蓋過人聲。
- **開播前等待 + 狀態延續**：播放前先經 `_wait_for_mic_idle`（含 `0.25s` 防抖）確認麥克風空閒才啟動 `afplay`/`say`；`always_on_mic` 狀態跨 chunk 傳遞，且只要偵測到麥克風轉為空閒就立即重設恢復保護。
- **CoreAudio 進程過濾（macOS 14.2+）**：透過 `kAudioHardwarePropertyProcessObjectList`（`prs#`）→ `kAudioProcessPropertyIsRunningInput`（`piri`）→ `kAudioProcessPropertyPID`（`ppid`）+ `proc_pidpath` 可精準識別特定 App（如 `Antigravity`）是否正在錄音，避免被背景 WebRTC/Siri 的全域 `dIn`/`gone` 狀態干擾。
- **預設策略**：`mic_process_filter` 預設 `[]`（所有 App 講話都會暫停），但偵測到 `Antigravity` 正在錄音時優先權最高——**永不因 `mic_wait_timeout`（預設 30 秒）超時搶播，也不會被背景常駐麥克風的 `always_on_mic` 忽略**；一旦 Antigravity 結束錄音便立刻開播或恢復。

## 輪次切換時的停止時機

- `hooks.json` 僅保留 `Stop`（`--speak`），不掛 `PreInvocation`（`--pre`）。
- 送出下一個 prompt 時不立刻砍掉前一輪語音，讓前一次回覆繼續播完（講話時仍會自動暫停）；直到下一輪回覆輸出完畢、`--speak` 準備開播新語音時，才由 `stop_existing()` 停掉前一次尚未播完的播放。
