---
title: HAPI compiled CLI 的 job run -- 參數正規化 bug
scope: tool
tool: hapi
tags: [hapi, session-jobs, bun, cli]
status: verified
created: 2026-08-24
updated: 2026-08-24
---

# HAPI compiled CLI 的 `job run --` bug

在 HAPI `0.29.0.1` standalone Bun compiled binary 上，以下官方語法無法啟動 child command：

```bash
hapi job run "$HAPI_SESSION_ID" smoke --label smoke -- /usr/bin/true
```

實際結果是 `--` 後的程式被當成預設 Claude command，出現：

```text
Error: Input must be provided either through stdin or as a prompt argument when using --print
[local]: Local Claude process failed: Process exited with code: 1
```

`hapi job list` 不會出現 job，child 也沒有執行。

## Root cause

`cli/src/utils/cliArgs.ts` 的 `normalizeCliArgs()` 會在 command dispatch 前處理第一個 `--`。Bun compiled argv 被判定為 runtime wrapper 時，只保留 `--` 後的 argv；所以 `resolveCommand()` 看不到 `job`，改走 default `claudeCommand`。`job.ts` 自己雖然需要 `--` 來分隔 child command，實際上收不到它。

`hapi codex`、`hapi cursor` 等是選擇新 HAPI session 的 agent backend，不是 `job run` 的 agent 參數，無法修正此問題。

## Safe handling

- 工作會在目前 agent turn 內完成：直接 foreground 執行並持續監看，不需要 attached job。
- 工作會 outlive agent：在修正 HAPI CLI 前，只能用官方 manual path，讓 wrapper 自己定期執行 `hapi job update` 並在退出時標記 `completed`/`failed`；不可用 set-once + nohup。
- 不要因 `hapi job run` exit 1 誤判 child command 失敗；先確認 child output、PID 與 `hapi job list`。
