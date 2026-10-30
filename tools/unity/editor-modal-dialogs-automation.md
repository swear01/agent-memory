---
title: AI 驅動 Unity Editor 時的阻塞對話框（Safe Mode、Scene(s) Have Been Modified）
scope: tools/unity
tool: unity
project: verto
status: active
created: 2026-10-31
updated: 2026-10-31
tags:
  - unity
  - unity-pipeline
  - automation
  - safe-mode
  - modal-dialog
---

# AI 驅動 Unity Editor 時的阻塞對話框

多個 AI session 各自在自己的 worktree 跑 GUI Editor、只透過 Unity CLI + `com.unity.pipeline` 操作時，Editor 跳出的 modal 對話框需要人點，會卡住 Pipeline，使用者也覺得很煩。使用者只看不改，所以預期行為一律是「不存檔、從磁碟重載」。

## 三種對話框（Unity 6000.6.7f1 執行檔內的字串）

- `Enter Safe Mode?`：開專案時有編譯錯誤就跳。常見於剛同步 main 或剛建好、Library 是從別的 commit 複製來的新 worktree。
- `Scene(s) Have Been Modified`（Save / Don't Save / Cancel）：場景有未存改動時，又被要求切換場景、跑 PlayMode 測試或重載就會跳。
- `The open scene(s) have been modified externally`：git merge、sync 或 checkout 在 Editor 開著時改寫了 `.unity` 檔。

## 處理方式

- 啟動時加 `-ignoreCompilerErrors` 跳過 Safe Mode。編譯錯誤照樣進 Console，Pipeline 仍可連上讀取：
  `open -g -j -n -a /Applications/Unity/Hub/Editor/6000.6.7f1/Unity.app --args -projectPath "$PWD" -ignoreCompilerErrors`
  這個參數名稱已在 6000.6.7f1 執行檔字串中確認（`ignoreCompilerErrors`）；Verto 的 `no-save-dialog` 分支正在做實機驗證。
- 已經卡在對話框的 Editor 不用去點：只結束自己 worktree 的 Editor 和 AssetImportWorker，再用上面的參數重開。
- Scene 對話框的根治方式：工具和測試不要留下 dirty scene，要保留的改動用程式明確存檔，外部改動直接重載。Verto 的修正在 `no-save-dialog` 分支，合併後寫進 `Documentation/Development.md`。
- 排查時先確認每個 session 是不是各開一台 Editor、各指向自己的 worktree（`ps -axo command | grep "MacOS/Unity -projectPath"`），以排除 Editor 共用造成的互相改寫。

## 已驗證的修正（Verto PR #46，`Assets/Phase/Editor/NoSavePrompt.cs`）

- `[InitializeOnLoad]` 在 `EditorApplication.update` 每個 tick 用反射呼叫 internal 的 `EditorSceneManager.ClearSceneDirtiness` 清掉 dirty；Play 中不動。實測 dirty scene + PlayMode `SandboxTests` 12/12 無對話框；沒這個 hook 時主執行緒卡在 `ShowAlertDialog`，Pipeline eval 全部 `Main thread operation timed out`。
- 外部改檔必須在 `update` 內輪詢 `.unity` 的 mtime、變了就 `OpenScene(path, Single)`。不能靠 `AssetPostprocessor`（匯入晚於原生檢查）、不能節流到 1 秒、不能在 `isUpdating` / `isCompiling` 時跳過：原生 `HandleOpenScenesChangeOnDisk` 在同一個 tick 觸發，三種做法都實測讓 Reload/Ignore 對話框搶先。
- 自己用 `SaveScene` 存檔時不能重載（會讓場景 handle 失效）：`sceneSaved` 記下當下 mtime，之後 mtime 一致就不動。
- 判斷是否卡在對話框：`sample <pid> 2` 看主執行緒堆疊有沒有 `ShowAlertDialog`；macOS accessibility 看不到被隱藏的 Editor 視窗。
- 剛開的新 worktree 用 `cp -cR` 的 Library 可能先編譯一次、套件還沒解析完而出現缺 `Keyboard` / `JObject` 的錯誤；用 `-ignoreCompilerErrors` 開不會跳 Safe Mode，重啟一次 Editor 即可。

