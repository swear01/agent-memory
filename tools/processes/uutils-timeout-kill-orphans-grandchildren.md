---
title: uutils timeout -s KILL 只殺直接子行程，孫行程變孤兒並繼續佔用資源
scope: tools/processes
tool: timeout (uutils coreutils)
status: active
created: 2026-10-27
updated: 2026-10-27
tags: [timeout, uutils, coreutils, orphan, flock, benchmarking, ubuntu]
---

# uutils `timeout -s KILL` 不會殺掉孫行程

2026-10-27 在 agentic-ic3-repro 實測（`timeout --version` 為 `timeout (uutils coreutils) 0.10.0`，系統預設的 `timeout` 不是 GNU 版）。

- `setsid -w bash -c 'timeout -s KILL 2 /usr/bin/time -o /dev/null sleep 79'`：回傳 124，但 `sleep 79` 仍在執行（`/usr/bin/time` 被殺，`sleep` 成為孤兒）。
- 同一指令改用 `-s TERM`：`sleep` 也一起結束。

## 後果

SecIC3 的 `run_one.sh` 以 `timed.sh timeout -s KILL --kill-after=30 3600 /usr/bin/time ... rIC3` 執行。逾時後結果已記為 TO（rc 124），但孤兒 rIC3 繼續跑好幾個小時，並保有 `timed.sh` 經 `exec` 繼承的 core flock，最後 16 顆 E-core 被佔 14 顆，其他 driver 全部排隊卡住；有一筆孤兒甚至在時限外印出 `RESULT: UNSAT`。

## 作法

- 讓計時器殺整個 process group：外層用 `subprocess.Popen(..., start_new_session=True)`，逾時或結束後 `os.killpg(pid, SIGKILL)`（agentic-ic3-repro `scripts/run_baselines.py` 就是這樣，因此沒有孤兒）。
- 或避免在 `timeout -s KILL` 與實際工具之間多夾一層 wrapper（如 `/usr/bin/time`）。
- 解析結果時，逾時 rc（124/137）要優先於 log 中的 `RESULT` 字樣，因為孤兒可能在時限外才寫入結果。
- 看到 core lock 長時間被佔，先用 `ps -eo pid,ppid,etime,cmd` 找 ppid 為 1 的工具行程。
