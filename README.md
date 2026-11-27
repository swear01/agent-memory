# Shared Agent Memory

這是跨 agents、projects、sessions 與 machines 共用的 persistent engineering memory。Markdown 檔案是唯一的 canonical memory source，內容可主要使用繁體中文並保留 English technical terms。

先記容易做錯的事。首段寫出情境、可辨認的症狀、正確做法，以及能證明結果的檢查。保留實際會改變下一步的版本、命令和例外。新證據改變做法時，更新原筆記並留下原始證據的連結。純結案、例行驗收和逐次量測留在原本的紀錄。有沒有要更新時，若沒有新的防呆經驗，就直接回覆「沒有需要更新的 QMD 記憶」。

## Hierarchy

- `global/`: 跨專案的 pitfalls、workflows、conventions、decisions
- `tools/`: pi、OpenCode、Codex、Cursor 等工具的行為與 workaround
- `domains/`: 可重複使用的技術領域知識
- `projects/`: 特定 repository 的知識
- `machines/`: 特定機器與環境的知識
- `archive/`: 已被取代的歷史記錄。獨立封存用 `.txt`，可避開各機既有的 Markdown 索引範圍。

QMD 是唯一的搜尋與索引引擎，日常搜尋指定 `-c memory`。QMD SQLite/vector index、models 與 cache 都是本機可丟棄的 cache，不提交到這個 repository。

GitHub repository `swear01/agent-memory` 是跨機器同步層；agents 應在重複調查前先搜尋相關 memory，並把已驗證且具長期價值的工程經驗寫回來。
