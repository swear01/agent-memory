---
title: Unity Pipeline 套件 (com.unity.pipeline) 與 Unity CLI 運作機制與即時控制
scope: tools/unity
status: active
updated: 2026-09-21
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
2. **模型替換與動畫縮放踩坑（Kenney FBX）**：
   - **動畫片段名稱檢查**：部分 FBX 首個 Take 為 `Root|0.Targeting Pose`（0.03s 靜態姿勢），需依名稱指定為 `Root|Idle`、`Root|Run`、`Root|Jump`，避免角色凍結在 T-pose。
   - **FBX 100 倍 Scale 關鍵幀**：Blender/3ds Max 導出的動畫常帶有根骨骼 Scale=100 曲線，在 Play Mode 下會使角色膨脹上百公尺。必須在 `ModelImporter` 開啟 `removeConstantScaleCurves = true`。
   - **腳本 Squash & Stretch 硬編碼**：控制器腳本（如 `Player.cs`）若硬編碼 `Vector3.Lerp(..., Vector3.one)`，會在運行時強制覆蓋 Prefab 的 localScale。應在 `Awake` 緩存 `m_InitialScale`，所有形變以 `Vector3.Scale(m_InitialScale, ...)` 相對縮放。
