---
title: 不要把工作檔寫進使用者家目錄
scope: global/workflows
status: active
updated: 2026-10-28
tags: [home-directory, cwd, scratch, cleanup]
---

# 不要把工作檔寫進使用者家目錄

## 問題

2026-10-28 在 swairM5 家目錄發現大量 agent 留下的雜物：`terminals/`（58 個指令紀錄，多數 `cwd` 是家目錄）、`agent-tools/` 工具輸出、打錯字產生的 `1echo`、整批 DVLab preview（mirror、zip、mongo、log、pid）、投影片中間渲染檔 `synopsys-<date>/`、`ta-preview/`、`.pytest_cache`、`.hapi-dev-test`、`.secrets.bak-*`。根因是 session 以家目錄為 cwd 啟動，agent 把暫存與輸出直接寫進目前目錄。

## 規則

- 使用者要求家目錄保持乾淨。不要在 `<remote-home>` 頂層建立任何檔案或資料夾。
- 暫存檔放 harness scratchpad 或 `/var/tmp`；專案檔放 `<remote-home>/Documents/<project>`；task worktree 放 `<remote-home>/.agent-worktrees/`。
- 以家目錄為 cwd 被啟動時，任何輸出都要先 `cd` 到適當目錄，或寫絕對路徑。
- 不要為一次性的 preview 建常駐 launchd 服務；做完就停掉並移除 plist。
- 不要留 `.secrets.bak-*` 類型的密鑰備份；使用者不需要。

## 清理做法

- 先分類：純垃圾、要搬進 Documents 的 repo、要保留的（`agent-memory/`、在 PATH 裡的 `bin/`、dotfiles）。
- Git repo 搬移前先檢查 dirty 狀態與分支是否已推上 origin；服務要先 `launchctl bootout` 再搬。
- 被視為「已備份」的檔案要先用 `cmp` 比對；本次的 m4a 和雲端版本不同（家目錄較新），所以改成放進雲端資料夾，沒有丟掉。
- Agent 只能移到垃圾桶；清空垃圾桶（永久刪除）由使用者自己做。
