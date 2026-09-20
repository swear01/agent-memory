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
- **檢查 Console 輸出**：
  ```bash
  unity command console --level warn
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
2. **模型替換時的尺寸核對**：
   - 替換不同來源的模型（如 Kenney FBX 人形模型替換原版機器人）時，網格原始尺寸可能差異甚大（如 3.96 單位 vs 1.10 單位）。
   - 可在 Unity 內利用 `unity command eval` 讀取 `Renderer.bounds.size` 比對 `CharacterController` 的 `Height` 與 `Center`，計算出正確比例（如 0.28）直接套用至 `character.localScale`。
