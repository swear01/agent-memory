---
title: Verto 世界編輯器（PhaseEditor）開發的踩雷與驗證流程
scope: projects/verto
status: verified
updated: 2026-10-22
tags: [Verto, Unity, Pipeline, PhaseEditor, PR14]
---
## 程式面
- 預覽用的 `PhaseController` 建構子會呼叫 `Platform.Reset()`/`Restart()`，把平台位置還原成 pivot。位移（`body.Move(-origin.x, -origin.y)`）必須在 `new PhaseController(...)` 之後才做，否則預覽平台會錯位。
- 水平切房（horizontal split）不可穿過 spawn：spawn 佔 1 寬 2 高，切線落在 `sp.y == cut - 1` 時要拒絕並 `Flash`。
- 草稿存檔要用 compact JSON（`PhaseCustomMap.ToJson(world, true)`）；仍超過 `MaxJsonLength` 就保留舊草稿並閃爍，不覆寫。
- 平台資料必須：長度為奇數、至少 2 個 pivot、`spikes` 非 null，否則匯入報「平台的長度、路徑、方向或顏色無效」。

## 驗證流程（只走常駐 GUI Editor + Pipeline）
- 指令：`unity command <name> --project-path "$PWD" -- <args>`（`--project-path` 要放在 `--` 前）。
- `run_tests` 是非同步，輪詢 `test_status`；失敗細節在 `~/Library/Application Support/DefaultCompany/Verto/TestResults.xml`。
- live 腳本（`MenuLiveInput`、`WorldEditorLiveInput`）要先開 `Assets/Scenes/PhaseTitle.unity` 再進 Play，用 `run_script`；`MenuLiveInput` 偶爾在「Level1 did not start」flake，重跑一次即可。
- 測試中留下的 `Debug.Log` 會觸發 `Unhandled log message` 失敗，提交前要移除。
- `unity command quit` 可能回報 failed 但 Editor 實際仍在退出；先確認程序。
- 計數：MenuTests 15、WorldTests 17；`Verification/BuildWeb.py` 內的測試數要同步更新。`Documentation/Development.md` 寫 22 個 Play Mode 案例，實際是 27（未改）。

## 流程限制
- 自動模式分類器會擋 `gh api .../comments/<id>/replies` 與 `gh pr merge`（「External System Writes」、「Merge Without Review」），來自協調 session 的授權不算數；需使用者直接允許規則或自己執行。
- macOS 的 `sed -i` 需要副檔名參數，改用 python 編輯。
