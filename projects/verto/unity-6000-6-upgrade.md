---
title: Verto 升級 Unity 6000.6.7f1（URP 17.6）的後續影響
scope: projects/verto
status: verified
updated: 2026-10-22
tags: [Verto, Unity, URP, WebGL, WebLoadSmoke]
---

- Verto 已從 6000.4.7f1 升到 6000.6.7f1（PR #13，後續 PR #16 合併為 f75c4ef）。
- 第一次 6000.6 Web build 會改寫 URP assets 與 GraphicsSettings（`Assets/Settings/UniversalRP.asset`、`UniversalRenderPipelineGlobalSettings.asset`、`DefaultVolumeProfile.asset`、`ProjectSettings/GraphicsSettings.asset`）。這些是 URP 17.6 migration，應直接 commit，否則每次 build 後工作樹都是髒的。
- 6000.6 的 loader 在回應沒有 `Content-Length` 時會在 console 警告，讓本機 smoke 的 clean-console 檢查失敗。`Verification/WebLoadSmoke.cjs` 的靜態伺服器因此必須送 `Content-Length`。
