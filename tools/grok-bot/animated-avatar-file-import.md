---
title: Grok Bot GIF 頭像的原生設定入口與 Mac 播放驗證
scope: tools/grok-bot
status: verified
created: 2026-10-09
updated: 2026-10-09
tags:
  - grok-bot
  - gif
  - avatar
  - macos
---

# Grok Bot GIF 頭像

## 已驗證的方法

2026-10-09 在 Mac Grok Bot 0.68.1 完成五個自訂 GIF 頭像設定，並確認 App 實際播放。

1. 使用已完成的循環 GIF；本次檔案約 1.1–1.3 MB，512×512、10 畫格、無限循環。
2. 請目標 Bot 將 Mac 原始 GIF 原封不動複製到它的雲端 `/workspace`。
3. 請 Bot 用內建的「檔案路徑設定自己頭像」功能設定該雲端 GIF，不經桌面裁切編輯器、不轉 PNG、不重新生圖。
4. 比對原檔與保存內容的 SHA-256，檢查 GIF 格式、畫格數與循環設定，再確認 Mac App 的實際動畫。

可用指示：「請把 `/workspace/avatar.gif` 原封不動用內建功能設為自己的頭像，不裁切、不轉 PNG，核對保存後格式、畫格數與 SHA-256。」

本次直接提供 Mac 檔案路徑首次回報 `Could not read`；先複製到雲端再設定成功。這是已重現的可用流程，不代表所有版本都無法直接讀 Mac 路徑。無須安裝生圖技能、修改 App、改資料庫或呼叫未公開 API。

## 桌面裁切入口為何變靜態

Mac 0.68.1 的頭像編輯器保存時將裁切內容畫到 256×256 canvas，執行 `toDataURL("image/png")`，透過 `setAgentAvatarBytes` 保存 PNG。即使候選預覽顯示 `.gif`，且確實按下「設定頭像」，保存結果仍可能只有單張 PNG。

原生選檔曾未載入 GIF 候選；Finder 複製 GIF 後在上傳區貼上可載入，但仍不能繞過編輯器保存時轉 PNG。未查明選檔未載入的根本原因，不把它推定為 GIF 不受支援。

Bot 內建入口保存的檔名可能叫 `avatar.png`，內容卻仍為 `GIF89a`；必須看實際位元組與解碼結果，不能用副檔名判定格式。Mac 的 `roster-avatars` 快取可保存原始 GIF 並播放。

## 完成證據與邊界

- 五個 Mac 頭像快取皆與對應原始 GIF 的 SHA-256 完全相同；GIF、512×512、10 畫格、loop=0。
- 擷取實際 App 畫面 85 次，歷時 7.548 秒；每個側邊欄頭像均捕捉到九個不同姿勢，並目視核對。
- 原始檔與證據保留於 `<task-root>/outputs/grok-action-avatars/`：`native-import-verification.json`、`native-five-app-playback.gif`、`native-five-motion-proof.png`、`native-avatar-research-2026-10-09.md`。
- 先前桌面編輯器轉 PNG 的紀錄只是該入口的限制，不能泛稱 Grok Bot 無法使用動態 GIF。
- 這些是固定循環 GIF，沒有接上思考、工作或閒置狀態切換；原生造型的狀態動畫能力不能推定適用自訂 GIF。
- 未看到同學本人操作；只核對公開作者的 Bot 自行設定方法並自行重現。未測其他 App 版本或手機端。

## 後續驗證原則

工具回報設定成功、頭像可見或檔名為 GIF，都不足以單獨證明動畫保存。先核對保存位元組，再對實際 App 連續取樣至少覆蓋最長循環；單張畫面或短時間恰好落在休息姿勢，不能據此判定靜態。
