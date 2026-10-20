---
title: Unity Pipeline 套件 (com.unity.pipeline) 與 Unity CLI 運作機制與即時控制
scope: tools/unity
status: active
updated: 2026-10-20
tags:
  - unity
  - unity-cli
  - unity-pipeline
  - automation
  - editor-scripting
---

# Unity Pipeline (com.unity.pipeline) 與 Unity CLI 即時控制

Unity 官方推出 Unity CLI 與 Unity Pipeline 模組（`com.unity.pipeline`），可取代傳統批次模式（`-batchmode` 需要重啟或鎖定專案）的終端自動化操作。

## 架構與核心組件

1. **Unity CLI**
   - 位於 `<remote-home>/.unity/bin/unity`。
   - 支援 `unity pipeline`、`unity command`、`unity status`、`unity open`、`unity close` 等原生指令。
2. **Unity Pipeline Package (`com.unity.pipeline`)**
   - 安裝於專案 `Packages/manifest.json`。
   - 指令：`unity pipeline install --project-path <project-root>`。
   - Unity Editor 開啟專案並載入該套件後，會在編輯器內部啟動本機通訊伺服器（預設 Port 7800）。
   - 狀態檢查：`unity status` 或 `unity pipeline list`。

## 即時指令控制 (Live Editor Command Execution)

當 Unity Editor 運行且 Pipeline 伺服器啟動（Port 7800 ready）時，終端可直接與 Editor 互動，無須關閉或重啟：

- **動態執行 C#（Roslyn Eval）**：
  ```bash
  unity command eval --code "return UnityEngine.Application.unityVersion;"
  ```
- **檢查 Console 輸出與截圖**：
  ```bash
  unity command console --level warn
  unity command screenshot --view game --output /tmp/game_play.png
  ```
- **取得可用指令清單**（包含 Prefab 建立、場景載入、序列化欄位修改等 150+ 種操作）：
  ```bash
  unity command
  unity command --query prefab
  ```

## 經驗與注意事項

1. **避免在 Editor 運行中直改 YAML 或啟動 Batchmode**：
   - 舊有做法在 Editor 開啟時執行 `-batchmode` 會因 Unity 實例鎖檔（`UnityLockfile`、`ArtifactDB-lock`）而失敗。
   - 手動編輯 YAML Prefab/Scene 容易在 Editor 獲得焦點時被覆蓋或造成同步問題。
   - 優先使用 `unity command eval`，在 live Editor 內透過 `PrefabUtility.LoadPrefabContents` 修改並以 `PrefabUtility.SaveAsPrefabAsset` 儲存。
2. **模型替換、朝向與動畫縮放踩坑（Kenney FBX）**：
   - **模型座標朝向反轉（-Z vs +Z）**：Blender 導出的 FBX 模型預設前方常為 `-Z`（如 Kenney 系列人形模型）。Unity 的標準前方是 `+Z`，角色控制器常透過 `Mathf.Atan2(velocity.x, velocity.z)` 將根物件朝向速度方向。若 Prefab 中的模型子物件為預設 `(0, 0, 0)`，角色的視覺面朝方向會完全相反（變成太空漫步、倒著走）。應在 Prefab 中將模型子物件的 `localEulerAngles` 設為 `(0, 180, 0)`，使視覺網格面朝與根物件 `+Z` 對齊。
   - **動畫片段名稱檢查**：部分 FBX 首個 Take 為 `Root|0.Targeting Pose`（0.03s 靜態姿勢），需依名稱指定為 `Root|Idle`、`Root|Run`、`Root|Jump`，避免角色凍結在 T-pose。
   - **FBX 100 倍 Scale 關鍵幀**：Blender/3ds Max 導出的動畫常帶有根骨骼 Scale=100 曲線，在 Play Mode 下會使角色膨脹上百公尺。必須在 `ModelImporter` 開啟 `removeConstantScaleCurves = true`。
   - **腳本 Squash & Stretch 硬編碼**：控制器腳本（如 `Player.cs`）若硬編碼 `Vector3.Lerp(..., Vector3.one)`，會在運行時強制覆蓋 Prefab 的 localScale。應在 `Awake` 緩存 `m_InitialScale`，所有形變以 `Vector3.Scale(m_InitialScale, ...)` 相對縮放。
3. **CLI 運行時截圖與背景更新 (`Application.runInBackground`)**：
   - 透過 `unity command editor_play` 進入 Play Mode 時，若 Unity Editor 處於背景，編輯器預設會停止刷新幀（`Time.frameCount` 停滯）。
   - 執行 `unity command eval --code "Application.runInBackground = true;"` 可使 Unity 在背景維持主循環更新，確保攝影機追蹤（Lerp）、物理模擬與動畫狀態正常運算，才能截取到真實 Play Mode 畫面。
4. **關閉 Editor（Pipeline 0.8.0-exp.1，Unity 6000.4.7f1，2026-10-20 實測）**：
   - `unity command quit -- --exitCode 0` 在 Edit Mode 失敗：`Command 'quit' failed: The following game object is invoking the DontDestroyOnLoad method: PipelineQuitScheduler`。
   - 隱藏、未聚焦的 Editor 不會執行 `EditorApplication.delayCall`，即使 auto-tick 開著，所以用 `eval` 排程 `delayCall += () => EditorApplication.Exit(0)` 不會關閉。
   - 可行做法：先用 `eval` 確認 `SceneManager.GetSceneAt(i).isDirty` 全為 false，再直接 `unity command eval --project-path <project-root> -- 'UnityEditor.EditorApplication.Exit(0); return "exiting";'`。回覆會中斷成 `Invalid response format from Pipeline server`，以行程結束確認成功。
5. **`--project-path` 必須放在 `--` 分隔符號之前（CLI 1.0.0-beta.13，2026-10-20 實測）**：
   - 正確：`unity command eval --project-path <project-root> -- '<code>'`、`unity command run_tests --project-path <project-root> -- --mode editor`。
   - 放在 `--` 之後會被當成指令參數，回 `Invalid arguments for set_autotick: set_autotick has no parameter --project-path <project-root>`，而且指令完全沒有執行；包成 shell 函式時特別容易犯（`unity command "$@" --project-path …`）。同時開多個 Editor（多個 worktree）時每個指令都要帶，否則可能打到別的專案。
6. **`run_tests` 是非同步的**：`unity command run_tests -- --mode editor --async_tests true` 立刻回 `0/0 passed (EditMode, 0s)`，那不是結果；要輪詢 `unity command test_status` 直到 `completed: N/N passed`。前一輪還在 `running` 時再送一次 `run_tests` 不會產生新的結果，要等前一輪完成。
7. **`eval` 主執行緒逾時不代表失敗**：長時間的 Editor 操作（例如建立並存兩個 scene、改 Build Settings）會回 `Pipeline server returned 400 Bad Request: Internal Server Error. Main thread operation timed out after 5000ms`，Console 也會記一筆 `Failed to handle /api/exec request`，但操作本身照樣完成。以實際產物（檔案、`EditorBuildSettings`、`isDirty`）確認，不要直接重跑造成重複。
