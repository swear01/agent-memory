---
title: Gamejam1 多 session 並行開發與 stacked PR 的經驗
scope: projects/gamejam1
project: gamejam1
status: active
created: 2026-10-20
updated: 2026-10-20
tags: [claude-code, sessions, worktree, stacked-pr, unity-pipeline, BuildWeb]
---

2026-10-19～20 在 Claude Code desktop 用多個 session（各自 worktree）並行做 PHASE 的開始畫面、遊戲內編輯器、鍵位/選關選單、捲動房間、關卡路線工具與 itch.io 發布時，實際遇到並確認的做法。

## 每個 worktree 一個 Unity Editor

- 每個 session 在自己的 worktree 用 `open -g -j -n -a <editor-app> --args -projectPath <worktree-root>` 開自己的 Editor，所有 `unity command` 都要帶 `--project-path <worktree-root>`，並放在 `--` 之前（見 tools/unity 筆記）。
- 同時開四個 Editor 可以正常各自跑 Pipeline 測試與 WebGL build；收工時要逐一確認 `isPlaying == false`、沒有 dirty scene，再 `EditorApplication.Exit(0)`。
- WebGL build 每次都會改寫 `Assets/DefaultVolumeProfile.asset`、`Assets/Settings/UniversalRP.asset`、`Assets/UniversalRenderPipelineGlobalSettings.asset`、`ProjectSettings/ProjectSettings.asset`（URP shader prefilter 等），與功能無關，commit 前要還原。

## 並行 PR 一定在測試數量上衝突

- `Verification/BuildWeb.py` 對每個測試套件寫死預期數量（例如 `("playmode", "MenuTests", 2)`），`Documentation/Development.md` 也列出測試數。任何兩個同時新增測試的 PR 都會在這兩個檔案衝突。
- 處理方式：依序合併，後合併的 PR 由原 session 把 main 併進來，兩邊數量都保留（相加），再重跑完整 BuildWeb 與 DemoWebSmoke。衝突本身簡單，但一定要重跑，因為新的測試（例如每關路線重播）會在另一個 PR 的引擎改動下執行。

## Stacked PR 合併

- 倉庫 `deleteBranchOnMerge` 為 false：合併底層 PR 後，上層 PR 的 base 不會自動改成 main；直接合併會只併進舊分支而沒進 main。先 `gh pr edit <n> --base main`（在底層合併之前改也可以，合併結果相同），再依序合併。
- 兩個 PR 疊在同一個 base 上、彼此不知道對方時，用 `git merge-tree --write-tree --name-only origin/<a> origin/<b>` 先預測衝突檔案，再決定合併順序。

## 其他

- Auto mode 下 `gh pr merge` 會被 classifier 以 Merge Without Review 擋下；使用者在設定加入允許規則後才可合併，不要繞過。
- session 因 `API Error: Can't reach the API server (ENOTFOUND)` 中斷時，工作區與背景程序都還在；送一則訊息請它檢查狀態後繼續即可恢復。
- 同學直接推到 main 的關卡改版（`904298c` 重做 `PhaseDemo12`）會讓錄好的通關路線失效，main 本身的 `SandboxTests` 與 DemoWebSmoke 隨即失敗；這種失敗與並行 PR 無關，應先修 main（重錄路線）再要求各 PR 全綠。
