---
title: SwairM5 用 Poco F7 當 Claude／ChatGPT 的 Wi-Fi 麥克風
scope: machines/swairM5
status: active
updated: 2026-11-21
---

# Poco F7 當 SwairM5 的麥克風

> **2026-11-21 起 BlackHole 已暫時停用**：使用者平常使用時嫌虛擬喇叭／麥克風太吵（雜），要求先移除。
> 驅動沒刪，搬到 `/Library/Audio/Plug-Ins/HAL.disabled/BlackHole2ch.driver`，`killall coreaudiod` 後系統只剩內建麥克風與揚聲器。
> 要恢復這套接法，先執行 `sudo mv /Library/Audio/Plug-Ins/HAL.disabled/BlackHole2ch.driver /Library/Audio/Plug-Ins/HAL/ && sudo killall coreaudiod`（不用重開機）。
> 同目錄的 `ParrotAudioPlugin.driver`（`com.apple.audio.ParrotAudioPlugin`）是 Apple 的，不要動。AudioRelay App 本身沒移除。

2026-11-20 用 Homebrew 安裝 Apple Silicon 版 AudioRelay 0.27.5，並透過 Homebrew 下載的已簽署安裝包裝 BlackHole 2ch 0.7.1。BlackHole 安裝完後重開機，macOS 才列出虛擬輸入與輸出。重裝時可用 `brew install --cask audiorelay blackhole-2ch`，BlackHole 需要管理員核准。

1. Poco F7 和 Mac 連上同一個 Wi-Fi。手機 AudioRelay 選「伺服器 → 麥克風」，允許錄音。
2. Mac AudioRelay 選「播放器」，把音訊裝置設為 `BlackHole 2ch`，再從伺服器清單點手機。若未自動出現，可在播放器輸入手機畫面上的 IP。
3. macOS「系統設定 → 聲音」的輸入選 `BlackHole 2ch`，輸出選外接螢幕 `AG326UZD`。這次要用的是 Claude／ChatGPT 語音輸入。
4. 在目標 App 的輸入框按麥克風，對手機說話，先看草稿是否出現文字。若要改回 Mac 內建收音，在 macOS 聲音輸入切回 MacBook Air 的麥克風。

已確認 AudioRelay 從 Mac 端連到手機的 Android 伺服器，播放器輸出和 macOS 預設輸入均選到 BlackHole，使用者確認螢幕喇叭有聲、語音已通。Claude 個別轉寫、鎖屏後持續收音及長時間穩定性未逐項測試。使用者最後要求保留連線。

[AudioRelay 官方 macOS 接法](https://audiorelay.net/docs/macos/use-your-phone-as-a-microphone-for-your-mac)
