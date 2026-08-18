# Shared Agent Memory

這是跨 agents、projects、sessions 與 machines 共用的私人 persistent engineering memory。Markdown 檔案是唯一的 canonical memory source，內容可主要使用繁體中文並保留 English technical terms。

## Hierarchy

- `global/`: 跨專案的 pitfalls、workflows、conventions、decisions
- `tools/`: pi、OpenCode、Codex、Cursor 等工具的行為與 workaround
- `domains/`: 可重複使用的技術領域知識
- `projects/`: 特定 repository 的知識
- `machines/`: 特定機器與環境的知識
- `archive/`: 已被取代的歷史記錄

QMD 是唯一的搜尋與索引引擎，collection 名稱為 `memory`。QMD SQLite/vector index、models 與 cache 都是本機可丟棄的 cache，不提交到這個 repository。

GitHub 私人 repository `swear01/agent-memory` 是跨機器同步層；agents 應在重複調查前先搜尋相關 memory，並把已驗證且具長期價值的工程經驗寫回來。
