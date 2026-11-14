---
title: oracle 資源小，Android build 和重運算移到遠端
scope: machines/oracle
status: active
updated: 2026-11-14
tags: [oracle, android, gradle, qmd, resources]
---

# oracle 資源小，Android build 和重運算移到遠端

oracle（hostname `swever`）是 aarch64、2 vCPU、11 GB RAM，同時跑 Vaultwarden、swear-review、cloudflared 和多個 HAPI session，load average 常在 3～4 以上。

2026-11-14 實測：
- 沒有 Android SDK（`ANDROID_HOME` 未設，也找不到 SDK 目錄）。Docker 裡只剩 vaultwarden 容器；使用者推測以前跑 Android build 用的容器已經關掉了。
- 在這台跑完整的 `qmd embed -c memory`，CPU 吃到 135%、跑了 20 分鐘還沒完成，而且和其他 session 的 vitest 搶 CPU，最後手動停掉。

使用者偏好：這台限制多，Android build/test 和重運算改到遠端跑。
- Android 專案（例如 gpt-voice-input）靠 GitHub CI 跑單元測試、lint、assembleDebug，不在 oracle 上裝 SDK。
- 在 oracle 寫記憶後只跑 `qmd update`；`qmd search`（BM25）立刻就搜得到。embedding 交給有 GPU 的 Mac 跑，見 [[qmd-vsearch-slow]]。
