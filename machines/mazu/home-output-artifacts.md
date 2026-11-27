---
title: Mazu 共用 home 的工具輸出位置
scope: machines/mazu
status: active
created: 2026-09-25
updated: 2026-09-26
---


HAPI runner 的 `--workspace-root` 限制可選工作區。工具以 session 的 cwd／命令的 `workdir` 解析相對路徑。排查 home 頂層新產物時先看實際工作目錄。

NeuroAbs 的 Pyverilog `VerilogParser`（含 `src/check.py`）預設 `outputdir="."`，PLY 會留下 `parser.out` 和 `parsetab.py`。指定專案內輸出目錄，或從專案目錄執行。ABC 可在啟動目錄留下 `abc.history`。

清理前檢查文件對產物的引用、工作樹 ownership 和未提交修改。`git status` 成功只證明副本的 `.git` 指標有可用的 Git metadata。用 `git worktree list` 和實際 metadata 路徑辨認已登記工作樹，保護 `~/.agent-worktrees/`。同時查 `~/research-archives/` 中同前綴的 bundles、tarballs 和校驗檔。保留研究文件仍引用的證據，確認保留理由與授權後才刪除對應副本或備份。

[原始驗證紀錄](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/machines/mazu/home-output-artifacts.md)
