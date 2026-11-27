---
title: Unity Pipeline 套件 (com.unity.pipeline) 與 Unity CLI 運作機制與即時控制
scope: tools/unity
status: active
updated: 2026-10-31
tags:
  - unity
  - unity-cli
  - unity-pipeline
  - automation
  - editor-scripting
---

## 先確認連到正確的常駐 Editor

專案 `Packages/manifest.json` 需載入 `com.unity.pipeline`，可用 `unity pipeline install --project-path <project-root>`。CLI 位於 `<remote-home>/.unity/bin/unity`。依使用者流程使用常駐 Editor，避免同專案的 batchmode 鎖衝突與開啟時手改 YAML 被 Editor 覆寫。Prefab 用 live Editor 的 `PrefabUtility.LoadPrefabContents` / `SaveAsPrefabAsset`。

- CLI 1.0.0-beta.13 的 `--project-path` 放在 `--` 之前，例如 `unity command eval --project-path <project-root> -- '<code>'`。放後面會當作 command 參數，可能回 `set_autotick has no parameter --project-path`。
- 多個 Editor 時，目標的 Pipeline 未 ready 可能由其他 Editor 回答，甚至作用在別的專案。先用唯讀 `editor_status`／`eval` 對回應 `data.target.projectPath`，不符不送突變指令，直至目標匹配。在每次指令後也核對實際 target。
- 用 `unity status`、`unity pipeline list` 或 `unity command editor_status`。Pipeline 0.8.0-exp.1 沒有 `unity command status`，它會回 `Command Not Found` 並污染 Console。待確認有哪些指令時用 `unity command` 或 `unity command --query prefab`。背景 Play 幀停住時可用 `Application.runInBackground = true`，再看 `Time.frameCount`／Game View 截圖確認已更新。

## 指令回覆不等於執行狀態

| 現象 | 查證與處理 |
| --- | --- |
| `run_tests -- --mode editor --async_tests true` 回 `0/0 passed` | 用同一目標輪詢 `test_status`，核對是本次測試且完成預期數量。前一輪 running 時不送下一輪。Verto 的 `--filter` 當時一次只接一個類別名，`A\|B` 回 0/0。 |
| `recompile_status` 回 `up_to_date` | 沒有需要重編的腳本時，可視為成功終態；不要只等 `completed`。 |
| `eval` 回 `Main thread operation timed out after 5000ms` | 讀檔案、`EditorBuildSettings`、`isDirty` 等，操作可能已完成。不要再送未對帳的寫入。 |
| `run_script` 搜尋逾時且後續 503 settling | CLI 停止等待，Editor task 仍可能在跑。不要送第二次；沿 process/Console/實際任務確認。Gamejam1 RouteTool.Search 的歷史恢復需正常關閉該 Editor 後再開，別重跑該專案的路線搜尋。 |
| `quit` 在 Edit Mode 報 `PipelineQuitScheduler`，`delayCall` 也不動 | Unity 6000.4.7f1／Pipeline 0.8.0-exp.1 實測。先確認目標 process、`isPlaying`、`isCompiling`、所有場景 `isDirty`。允許關閉且資料已保存時，在該 Editor 直接 eval `UnityEditor.EditorApplication.Exit(0)`。回覆可能是 `Invalid response format from Pipeline server`，以 process 消失為證據。 |

## 遊戲執行與匯入

- Kenney FBX 子網格面向 -Z 時，將該子物件旋轉 Y 180° 對齊根物件 +Z。依片段名選 Idle/Run/Jump，別取首個 `Root|0.Targeting Pose`。模型播放膨脹時查根骨骼 100 倍 scale 曲線，`ModelImporter.removeConstantScaleCurves = true`；Squash/Stretch 以 Awake 保存的初始比例做相對縮放。
- WebGL 短按可能落在兩個 Update 之間，選單用 `wasPressedThisFrame` 讀單次觸發，連續移動另做按住 repeat。
- 常駐 Editor 反覆 Play 出現 `There are N audio listeners in the scene` 時，查 `HideFlags.HideAndDontSave` 的執行期物件殘留。Verto 以 `HideFlags.HideInHierarchy` 讓對象隨 Play 結束回收，重開 Editor 清既有殘留，再重複 Play 驗證。

[完整原始用法與專案驗證](https://github.com/swear01/agent-memory/blob/9b2d556e370f3a71736d5dc0a63f3b1e1c8c3a0e/tools/unity/unity-pipeline-cli-automation.md)
