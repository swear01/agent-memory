---
title: HAPI 0.29.0.1 job run 會遺失 command 分隔符
scope: tools/hapi
tool: hapi
tags: [hapi, session-job, cli, bun, argv]
status: verified
created: 2026-09-02
updated: 2026-09-02
---

# 症狀

HAPI `0.29.0.1` 執行下列命令時，child command 沒有啟動，反而落入預設 Claude launcher：

```text
hapi job run <session> <key> --label <label> -- /usr/bin/true
Error: Input must be provided either through stdin or as a prompt argument when using --print
[local]: Local Claude process failed
```

`strace -f -e execve` 證實原始 argv 含 `job run ... -- /usr/bin/true`，但後續實際啟動的是
`claude ... -- /usr/bin/true`。`hapi job set` 可正常寫入，故不是 hub token 或 job API 權限問題。

# 根因與處理

`0.29.0.1` 的 `cli/src/utils/cliArgs.ts::normalizeCliArgs` 會把第一個 `--` 移除；Bun compiled
binary 的 runtime-wrapper 判斷又可能把 `--` 前的 `job run` 一起丟掉，最後 command 被當成預設
Claude arguments。`0.29.0.3` 已加入 `normalizedPre[0] === 'job' && normalizedPre[1] === 'run'`
特例，保留 child separator。

不需要為單一 task 升級或重啟 fleet。可從 `swear01/hapi` release 下載
`hapi-linux-x64-baseline.tar.gz` 與 `checksums.txt` 到 `mktemp -d` 目錄，先用
`sha256sum --check --ignore-missing` 驗證，再以該臨時 `0.29.0.3` binary 執行 `job run`。
先跑 `/usr/bin/true` smoke，確認 job terminal status 為 completed 後再跑長工作。
