---
title: "ChatGPT 與 Codex 語音偏好的指示入口及驗證界線"
scope: tools/codex
status: verified-with-limits
updated: 2026-10-12
---

# 自訂指示的入口

桌面 App 與網頁版的「設定 → 個人化」有兩個不同入口：

- **編輯 ChatGPT 自訂指示**：帳號的 ChatGPT 對話偏好，另有暱稱、職業及個人介紹欄位。
- **編輯 Codex 自訂指示**：另一套 Codex 指示。桌面 Codex 的本機全域 `AGENTS.md` 也是獨立來源，不能用改寫它代替更新 ChatGPT 帳號設定。

2026-10-12 已透過內建瀏覽器完成 ChatGPT 自訂指示儲存，出現成功通知，重新載入後讀回內容一致。內建瀏覽器的登入狀態與桌面 App、外部瀏覽器不同；遇到未登入時交由使用者完成登入，再繼續。

修改前先等欄位載入完成，保留完整原文；空白欄位應重新載入確認後再寫入。只追加授權內容並保留其他欄位，儲存後重新載入讀回核對。單次讀到空字串不足以證明更早沒有別的版本；不要宣稱已排除所有覆蓋可能。

# 語音指示的作用與限制

可用簡短偏好表達允許重疊、簡短接話、區分附和與要求停下。保存偏好成功不等於已證明語音會主動插話；本次沒有進行實際語音行為測試。

先區分完整語音對話與 dictation 聽寫，也不要把 Chat 模式標籤直接視為目前使用的語音模型證據。

# 桌面 App 靜態稽核快照

以下只適用於已檢查的 App 26.1002.52244、build 13536，後續版本必須重驗：

- 原生語音的 WebRTC 麥克風輸入與接收音訊播放分離；檢查到的音訊 worklet 在即時階段持續傳入輸入，不會僅因助理說話而自動靜音。這是程式結構證據，並非現場雙向音訊測試。
- 語音基礎 prompt 由遠端設定提供。原生路徑的設定識別為 `1193530394`、欄位為 `prompt`；本機 fallback 不代表帳號當下使用的遠端 prompt。
- 該發行版的 `Hmi` 呼叫 `Umi` 時帶入 `canUseLocalOverride: false`，因此不能只寫入 `realtime-voice-config-override` 就聲稱成功覆寫 prompt。
- 查到的 `silence_duration_ms: 500` 屬於 dictation streaming，不能套用為完整即時語音的發話設定。
- 未擷取使用中遠端 session 設定，也未驗證帳號自訂指示對主動插話的實際效果。

官方參考入口：`learn.chatgpt.com/docs/features/voice`、`learn.chatgpt.com/docs/personalize`、`developers.openai.com/api/docs/guides/live-prompting`。API 的 full-duplex 與 prompting 能力不能直接當作桌面 App 已開放相同設定的證據。
