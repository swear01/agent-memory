---
title: "Android 長按選單的 AI 項目無法用 ADB 單獨移除"
scope: tools/android
status: active
updated: 2026-11-21
---

# Android 長按選單的 AI 項目無法用 ADB 單獨移除

長按選字後出現的 Ask GPT、Ask Claude、Ask Grok、Reddit Search 等項目，來自各 App 註冊的 `android.intent.action.PROCESS_TEXT` activity。在 POCO F7（HyperOS 3、Android 16）上實測：

- 列出來源：`adb shell cmd package query-activities --brief -a android.intent.action.PROCESS_TEXT -t text/plain`。
- 只關單一 activity 會失敗：`pm disable --user 0 <pkg>/<activity>` 回 `SecurityException: Shell cannot change component state`，第三方與系統 App 都一樣。Shizuku 以 shell 身分執行，推測同樣被擋。
- 停用整個系統 App 也會失敗：`pm disable-user` 回 `Cannot disable system packages`（Xiaomi 限制）。
- 可行的是 `pm uninstall -k --user 0 <pkg>`，能用 `cmd package install-existing <pkg>` 還原。Read aloud 來自 `com.google.android.marvin.talkback` 的 SelectToSpeakPopupActivity，用這個方式移除後就從選單消失。Claude Code auto mode 會擋這個指令，要使用者明確同意移除系統 App。
- Web Search 是 Brave/Chromium 自己加的；Ask Gemini 來自系統 Google App（`com.google.android.googlequicksearchbox`），移除會連帶影響搜尋與 Circle to Search。

不 root 又要保留 App 時，唯一候選是把 App 搬進工作資料夾（Island/Shelter），因為選單只查同一 user 的 handler。這只是推論，尚未實測；使用者最後決定不處理。

ADB 連線在這台手機上會不定時斷線，指令前加 `adb wait-for-device` 與 `timeout`，避免 `adb exec-out screencap` 卡住。截到的選單畫面是白的，看不到項目。
