---
title: Unity Licensing Client 以 LaunchAgent 常駐，避免多 Editor 同時失去授權連線
scope: tools/unity; SwairM5; 多個 Unity Editor 並行（Verto agent sessions）
status: active
updated: 2026-10-26
---

- 症狀：多個 Editor 同時出現 "Connection to Unity license client" 失敗。根因：Editor 啟動時若 `/tmp/Unity-LicenseClient-swear.sock` 不存在，會自己從 `Unity.app/Contents/Helpers/UnityLicensingClient.app` 啟動 client，該 client 隨該 Editor 結束而消失，其他連到它的 Editor 一併掉線（2026-10-25 19:19 UTC 殺掉擁有 client 的 Editor 觸發）。
- 修法：`~/Library/LaunchAgents/com.swear.unity-licensing-client.plist` 以 Hub 內的 `Unity Hub.app/Contents/Frameworks/UnityLicensingClient_V1.app/Contents/MacOS/Unity.Licensing.Client --namedPipe Unity-LicenseClient-swear` 執行（RunAtLoad、KeepAlive、`ThrottleInterval=1`、`ProcessType=Background`）；Hub 版 bundle 設 LSUIElement/LSBackgroundOnly，無 Dock 圖示。備份於 transfer_MAC `config/snapshot/home/Library/LaunchAgents/`。
- client 沒有常駐選項（`--help` 只有 pipe／entitlement 相關旗標）：最後一個 session 結束即自行退出（log 「Last active session ended」「exit timer」），由 launchd 重啟，throttle 1s 時空窗約 1 秒（實測 kill 後 ~1.5s 回來）。空窗內啟動的 Editor 仍會自己 spawn 一個 client，但該 client 在其 session 結束後退出、launchd 版自動接回；殘餘風險是別的 Editor 在空窗內連上那個 Editor-owned client。
- 驗證（GUI Editor、batchmode）：Editor 連到 launchd 版 client（ppid 1）；Editor 結束後 client 重生。
- "Access token is unavailable; failed to update" 是 Editor 端沒有 Hub 傳入的 access token，無害：同一份 log 之後 resolve entitlement 與 update license 均成功（Unity Personal）。client 端的 "401 Token not found in cache" 同理。
- Hub 更新若搬動 client 路徑，需更新 plist。復原：`launchctl bootout gui/$(id -u) ~/Library/LaunchAgents/com.swear.unity-licensing-client.plist && rm ~/Library/LaunchAgents/com.swear.unity-licensing-client.plist`。
- Unity 官方文件（docs.unity.com licensing ClientConfig）只記載 services-config.json 的逾時鍵，未記載 `--namedPipe` 與閒置退出。
