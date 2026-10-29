---
title: 等待迴圈的 pgrep -f 會比對到自己
scope: tools/shell
status: active
updated: 2026-10-29
tags: [pgrep, ssh, watcher, background]
---

# 等待迴圈的 `pgrep -f` 會比對到自己

`until ssh host '! pgrep -f "bench3.py gpu gpu6"'; do sleep 300; done` 這類等待迴圈永遠不會結束：遠端 `bash -c` 自己的命令列就含有那串字，`pgrep -f` 一定找得到自己。2026-10-29 主 session 有 6 個這樣的 waiter 卡了好幾個小時，RTL session 的 NVDLA watcher（`pgrep -f "yosys -l nvdla3"`）在 yosys 早上 07:03 結束後白等了 13 小時，ABC watcher 也因同樣原因沒回報。

做法（擇一）：

- 用字元類別讓樣式不等於字面字串：`pgrep -f "[b]ench3.py gpu gpu6"`。
- 程式名精確比對：`pgrep -x gpuls`。
- 直接等 PID：啟動時記下 `$!`，之後 `while kill -0 $pid 2>/dev/null; do sleep 60; done`（遠端則 `ssh host "while kill -0 $pid; do sleep 60; done"`）。
- 等輸出檔：完成時 `touch done.flag`，迴圈等那個檔。

waiter 超過預期時間還沒回報時，直接去看目標程序和輸出檔，不要假設它還在跑。
