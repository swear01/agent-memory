---
title: Parallel HAPI test runs need a private TMPDIR
scope: tools/hapi/parallel-test-tmpdir
project: hapi
status: active
updated: 2026-10-29
tags: [hapi, testing, vitest, worktree]
redaction: passed
---

## 每次執行使用獨立 TMPDIR

同時跑兩套 HAPI 測試會共用 `hapi-test-config.json`，一個 `globalSetup`／teardown 覆蓋或刪除它後，另一套 CLI 可能報 `Missing isolated hub config`。用 `TMPDIR=<existing-per-invocation-tempdir> bun run test`，每次 invocation 都用不同已建立的目錄，包含同一 worktree 裡的 focused 與 full suite。確認路徑、錯誤時間與 teardown 再判定碰撞，不宜把其他測試失敗都分類為環境。

## 只跑有隔離資源的 integration

一般測試不可因本機 Hub 剛好在線就啟用 integration suite。由明確的模式選擇和隔離 Hub/home 控制啟用，正常 suite 略過時如實說明覆蓋。這來自歷史競態診斷，沒有修正後整套通過證據。共用小主機過載時按當次耗時與執行器時間限制選 focused／full gate，不沿用歷史 elapsed。

[TMPDIR 復現](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/hapi/parallel-test-tmpdir.md) · [Hub 隔離案例](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/hapi/integration-test-admission.md)
