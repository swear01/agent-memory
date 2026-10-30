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
