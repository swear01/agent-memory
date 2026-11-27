---
title: HAPI 0.29.0.1 job run -- 誤啟動 Claude
scope: tools/hapi
tool: hapi
tags: [hapi, session-job, cli, bun, argv]
status: verified
created: 2026-09-02
updated: 2026-11-28
---

HAPI standalone Bun binary `0.29.0.1` 的 `hapi job run … -- …` 若顯示 `Input must be provided either through stdin or as a prompt argument when using --print`，先查 `hapi job list` 與 child 的 PID 或輸出。這個組合已觀察到誤啟動預設 Claude，原本的 child 未執行。若 `hapi job set` 成功，當時故障在 CLI 命令派發。

- 根因是 `cli/src/utils/cliArgs.ts` 的 `normalizeCliArgs` 誤把 `--` 前的 `job run` 當作 Bun wrapper 參數丟掉。`0.29.0.3` 已驗證保留分隔符。更換成 `hapi codex` 或 `hapi cursor` 只會選擇新 session 的 agent。
- 只為單一工作處理時，可用經 `checksums.txt` 核對的臨時修正版 binary，避開重啟整個 fleet。先跑 `hapi job run <session> <key> --label smoke -- /usr/bin/true`，確認該 job 到達 `completed` 後才跑長工作。
- 必須留在受影響版本時，短工作可前景執行並監看。跨 turn 的 attached job 需要 wrapper 定期 `hapi job update`，退出時標記 `completed` 或 `failed`。

[原始故障紀錄](../../archive/tools/hapi/job-run-compiled-dashdash-bug.txt)
