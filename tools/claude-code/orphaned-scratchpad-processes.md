---
title: "Mac 卡頓：Claude session 留下的 scratchpad 孤兒程序吃光記憶體"
scope: tools/claude-code
machine: SwairM5
tags: [macos, memory-pressure, swap, orphan-process, scratchpad]
status: active
created: 2026-10-26
updated: 2026-10-26
---

# Mac 卡頓：scratchpad 孤兒程序吃光記憶體

2026-10-26 在 SwairM5 上，電腦卡頓的根因是一個 `Python reach.py` 已經跑了 3 天多。
它是某個早已結束的 Claude Code session 在 scratchpad（`/private/tmp/claude-501/<project>/<session>/scratchpad/sim`）啟動的模擬腳本。
PPID 為 1（被 launchd 收養），`top` 顯示 state `stuck`。
當時實體記憶體只剩 124M、swap 用到 29.4/30G，`kernel_task` 吃超過 100% CPU。
結束這個程序後，可用記憶體馬上回到約 7.5G，壓縮區從 12G 降到 5G。

## 排查步驟

```bash
top -l 2 -n 15 -o cpu -stats pid,command,cpu,mem,state   # 也可改用 -o mem
sysctl vm.swapusage; memory_pressure | tail -1
ps -o pid,ppid,etime,rss,command -p <pid>                 # PPID 1 + etime 好幾天 = 孤兒程序
lsof -a -p <pid> -d cwd -Fn                               # cwd 在 claude-501/.../scratchpad 就是 agent 留下的
```

## 注意

- `top` 的 MEM 欄是 phys footprint，會把被壓縮和被換到 swap 的部分算進去（當時顯示 61G），`ps` 的 RSS 只有約 1G。判斷是不是元兇要看 footprint 和 swap，不能只看 RSS。
- 確認 cwd 和用途之後，先問使用者再 kill；先送 SIGTERM，沒結束才用 `-9`。
- 預防：agent 在 scratchpad 跑長時間模擬時要設 timeout，或在結束 session 前確認已經收掉（可參考 [[python/capture-process-groups]]）。
