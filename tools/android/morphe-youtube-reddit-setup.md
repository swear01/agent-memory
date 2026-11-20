---
title: "Morphe 取代 RVX：在 POCO F7 上修補 YouTube、Reddit 並搬移設定"
scope: tools/android
status: active
updated: 2026-11-21
---

# Morphe 取代 RVX：在 POCO F7 上修補 YouTube、Reddit 並搬移設定

2026-11-21 在 POCO F7（HyperOS、Android 16）上把 RVX 換成 Morphe 時，實際驗證過的做法與陷阱。

## 現況

- RVX（inotia00）2025-12-31 起停更；Morphe 由原 RVX 開發者維護，GitHub 組織是 `MorpheApp`。
- 手機目前裝的：`app.morphe.manager`（由 Obtainium 更新，有追 pre-release）、`app.morphe.android.youtube`、`com.reddit.frontpage`（Morphe 修補版）、MicroG RE（套件名仍是 `app.revanced.android.gms`）。
- Manager 會指定原始 APK 版本（這次是 YouTube 21.30.216、Reddit 2026.40.0），從 APKMirror 下載，核對頁面上的 SHA-256。

## 安裝

- `adb install` 會回 `INSTALL_FAILED_USER_RESTRICTED`，原因是 Xiaomi 的「USB 安裝」限制。可行做法：`adb push` 到 `/sdcard/Download`，用 `am broadcast -a android.intent.action.MEDIA_SCANNER_SCAN_FILE -d file:///sdcard/Download/<file>` 讓系統索引，再到 Google Files 點檔案安裝（Files 已有安裝權限）。
- APKMirror 用 curl 抓會被當成繞過 Cloudflare；改用內建瀏覽器點下載，檔案會落在 `~/Downloads/.<random>.com.anthropic.claudefordesktop.*` 暫存名。內建瀏覽器擋 reddit.com。
- Reddit 補丁版沒辦法用 Google 登入，App 內嵌的 WebView 會停在 `accounts.google.com` 白屏。要在一般瀏覽器處理，或改用 Reddit 帳號密碼登入。

## 設定匯入（Morphe YouTube）

- 匯入、匯出的 JSON key **沒有** `morphe_` 前綴（`Setting.getImportExportKey()` 會去掉）。帶前綴匯入會整批不生效。
- 匯入時，JSON 沒列到的設定都會被重設成預設值。所以要先匯出目前設定、合併後再整份匯入。
- 匯出只會列出非預設值，看不到所有選項；完整清單要看 `MorpheApp/morphe-patches` 的 `extensions/youtube/.../settings/Settings.java`。
- RVX key 多半要改名，例如 `hide_navigation_shorts_button` → `hide_shorts_button`、`default_video_quality_wifi` → `video_quality_default_wifi`、`hide_category_bar_in_*` → `hide_filter_bar_in_*`。約 18 項沒有對應設定。
- 投放按鈕有兩個開關：`hide_toolbar_cast_button`（首頁工具列）和 `hide_cast_button`（播放器）。
- 改圖示（`custom_branding_icon`）後，啟動入口會從 `.morphe_black_1` 換成 `.morphe_original_1`。
- 設定備份在使用者 Google Drive 的 `document/Setting Backup/app setting/`。

## ADB 自動化

- USB 連線會頻繁斷線，改成 Wi-Fi ADB 並設 `ANDROID_SERIAL=<ip>:5555` 後就穩了。每個 adb 指令都要加 `timeout`。
- 點擊前先用 `dumpsys activity activities | grep topResumedActivity` 確認 App 在前景，不然點擊會落在桌面。
- 同一支手機若有其他 agent 也在用 ADB，會互相搶前景；用 `logcat | grep "adbd service requested"` 可以看到對方下的指令。
- YouTube 畫面用 `screencap` 截出來是白的，改讀 `uiautomator dump`。

## Reddit 帳號

這次 Reddit 帳號被判定有「automated activity」並遭永久停權，通知早在裝補丁版之前兩天就出現了。使用者自行驗證、申訴後已恢復。不要讓腳本或 AI 瀏覽器代理登入這個帳號。
