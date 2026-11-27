---
title: Gamejam1 PHASE Web preview and live Pipeline CI
scope: projects/gamejam1
status: verified-public-preview-owner-ci-setup-pending
updated: 2026-10-20
tags: [Gamejam1, PHASE, Unity, WebGL, Unity-CLI, Pipeline, CI, Pages]
---

## 驗證與發布

- **Unity CLI 遙控成功不等於 Player 有跑。** 用 resident GUI Editor + 官方 Unity CLI / `com.unity.pipeline`，已驗證組合為 Editor 6000.4.7f1／CLI 1.0.0-beta.13／Pipeline 0.8.0-exp.1。Pipeline domain reload 失敗可能留 Play 在跑，先確認退出 Play，再編輯 C#。驗實際鍵盤輸入、前進幀與 Game View 截圖。保留工作區 dirty gear/wiki；關自己擁有的 Editor。
- **CI artifact 在 Editor 結束後不見。** `Verification/BuildWeb.py` 走 live Pipeline，輸出放 `TestResults/Web`；Unity 退出會清 `Temp`。WebGL build 可能改 UniversalRP、UniversalRenderPipelineGlobalSettings、DefaultVolumeProfile 及 ProjectSettings，先核對並還原屬於建置的修改，提交別用 `git add -A`。
- **Hosted controller CI 綠了，但遠端沒有 Unity build。** Web job 仍依 owner 設好授權 macOS ARM64 GUI runner（`gamejam1-unity`）、Actions Pages 與 `UNITY_WEB_CI_ENABLED=true`。依 `Documentation/Web.md`，維持 private repo；WRITE 不代表有 owner 權限。手動公開的 [Sites 預覽](https://phase-gamejam1-20261016.swear01.chatgpt.site/) 由原 checkout 的 `.openai/hosting.json` 決定站點，不另建站。正式 itch 是 [phase](https://swear01.itch.io/phase)，勿覆蓋歷史 HTML [phase-demo](https://swear01.itch.io/phase-demo)。每個發布入口和最新 CI 分別驗證。
- **瀏覽器 loader 完成、但畫面或按鍵失效。** Playwright 用 `channel: chromium`，等真正像素再送操作。檢查原生選關、勝利、replay、雙向換房、自訂地圖、重綁、fullscreen、load failure 與多種長寬比。像素用小容差和場景佈局判定；慢機器先 isolate 問題房間，勿放寬遊戲判定。公開 proxy 插入單一 challenge script 時分開記 raw match 與 product equivalence；HTTP 304／ERR_ABORTED 只有能和成功 UnityCache revalidation 一對一關聯時才排除。公開十二關已用匿名 Chromium 驗證，Safari、實體手機與人類平衡未測。
- **編輯草稿 reload 可讀，卻有 JS_FileSystem_Sync 警告。** 在 `createUnityInstance` 設 `autoSyncPersistentDataPath: true`，測真實 reload；Unity 6000.4.7f1 的 `PlayerPrefs.Save()` 手工同步會觸發警告。WebGL SNORM probe 只修 canvas-scoped 查詢，不壓 Console；Renderer2D 未用後處理可關。唯一維護的地圖編輯器是遊戲內 `PhaseEditor`，舊 HTML 編輯對話框不再擴充。

## 保留的產品行為

- **畫面極窄或極寬就裁地形。** 各長寬比至少可見原生 320×180。寬取 `max(320, round(180*w/h))`，高取 `max(180, ceil(320*h/w))`，置中原始玩法，延展合適背景與接地邊界，保持 point/pixelated 採樣。玩法畫面零解說文字；重綁 dialog 可在畫面外。
- **重綁用錯舊 Esc 邏輯。** 每動作兩個 slots，保留至少一鍵，左右 Shift 屬同一邏輯鍵。capture 中 Esc 清 slot、Tab 取消；之外 Esc 關閉設定。`phase.controls.v2` 從舊單鍵資料遷移首 slot，保留 v1。settings 暫停 controller 並清 queued input，關閉等 held keys 放完。核對 browser local storage 失敗可繼續玩。
- **房間邊界或危害的遷移遺失資料。** 只開有連結的左右邊，death 優先，過房保留原色、位置高度、速度及動作狀態，移除舊 contact。場景同時進 Build Settings 和 BuildWeb.py。mask0 保留 legacy phase，顯式 R1/B2/G4/Y5/M3/C6/W7 覆寫；PhaseSpike 長度依介面區分 cell 或 native pixel。swept carry 有 hazard 就停止移動。平台位移分段、獨立 queue/return，保留向下時有固定地板支撐的共享 rider。Visual 規則以 `DESIGN.md` 和 repo 的 frozen art/trace fixtures 為準。

[完整原文](https://github.com/swear01/agent-memory/blob/06f50e3a42e850b37cffb6b6a4a446ef670e9625/projects/gamejam1/web-preview-ci.md)
