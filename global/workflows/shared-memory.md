---
title: Shared persistent agent memory workflow
scope: global
status: active
created: 2026-08-18
updated: 2026-08-18
tags:
  - shared-memory
  - qmd
  - github
---

# Shared persistent agent memory workflow

這個 repository 是跨 agents、projects、sessions 與 machines 共用的 persistent engineering memory。Markdown 檔案是 canonical source。

使用 QMD 的 `memory` collection 搜尋與讀取記憶；不要把 QMD 的 SQLite/vector index 當成 canonical data。

私人 GitHub repository `swear01/agent-memory` 負責在不同 machines 之間同步這些 Markdown 記憶。新增或更新已驗證的長期經驗後，先 `git pull --rebase`，再 commit、push，並重新執行 QMD update/embed。
