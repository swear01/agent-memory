---
title: Mazu 共用 home 的工具輸出位置
scope: machines/mazu
status: active
created: 2026-09-25
updated: 2026-09-25
---

HAPI runner 的 `--workspace-root` 是 `<remote-home>`，只決定可選工作區的範圍；實際 Codex session 可以在專案目錄。若 agent 從 home 執行會寫相對路徑的工具，產物便落在 home 頂層。先檢查 session 的 cwd 和命令的 `workdir`，不要只看 runner 的 workspace root。

NeuroAbs 內的 Pyverilog `VerilogParser` 預設 `outputdir="."`，PLY 因而在執行目錄產生 `parser.out`、`parsetab.py`。`NeuroAbs/src/check.py` 的 parser 也使用相同預設。執行相關測試時指定專案內的輸出目錄，或至少從專案目錄執行。Berkeley ABC 在 home 執行後也可能留下 `abc.history`；2026-09-25 盤點時該檔為空。

清理臨時目錄前先檢查引用：`NeuroAbs/results/i2c_assert1/README.md` 直接引用 `<remote-home>/pono-dynamic-coi-sat.03GNm1/`，該目錄是研究證據。`<remote-home>/.agent-worktrees/` 是依工作流程集中放置的 Git 工作樹，必須先查 Git 狀態與 ownership，不能整批刪除。
