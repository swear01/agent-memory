---
title: Mazu 共用 home 的工具輸出位置
scope: machines/mazu
status: active
created: 2026-09-25
updated: 2026-09-26
---

HAPI runner 的 `--workspace-root` 是 `<remote-home>`，只決定可選工作區的範圍；實際 Codex session 可以在專案目錄。若 agent 從 home 執行會寫相對路徑的工具，產物便落在 home 頂層。先檢查 session 的 cwd 和命令的 `workdir`，不要只看 runner 的 workspace root。

NeuroAbs 內的 Pyverilog `VerilogParser` 預設 `outputdir="."`，PLY 因而在執行目錄產生 `parser.out`、`parsetab.py`。`NeuroAbs/src/check.py` 的 parser 也使用相同預設。執行相關測試時指定專案內的輸出目錄，或至少從專案目錄執行。Berkeley ABC 在 home 執行後也可能留下 `abc.history`；2026-09-25 盤點時該檔為空。

清理臨時目錄前先檢查引用：NeuroAbs 的兩組研究證據已搬到 `<remote-home>/research-archives/agent-cleanup-20260926/neuroabs/`，`NeuroAbs/results/i2c_assert1/README.md` 的引用由 PR #11 更新。`<remote-home>/.agent-worktrees/` 是依工作流程集中放置的 Git 工作樹，必須先查 Git 狀態與 ownership，不能整批刪除。

`issue174_repro` 的四個內層 `.git` 檔雖可讓 `git status` 執行，卻全指向同一個別處工作樹的 Git metadata，且未列在 `git worktree list`；它們是複製快照，不是四個已登記工作樹。Issue #174 已完成，實驗結論保存在 `projects/cpachecker/mathsat-crash-issue174.md`；2026-09-26 刪除了這份 3.1 GB 原始重現資料。

刪除 agent 建立的專案副本時，同時盤點 `<remote-home>/research-archives/` 中同名前綴的 Git bundle、tarball、清單與校驗檔；備份的保留理由若已消失，就在同一次清理中移除。2026-09-26 刪除 `pono-llm` 後曾漏查舊備份，後來補刪；目前這個 archive 只保留 NeuroAbs 文件仍引用的結果。
