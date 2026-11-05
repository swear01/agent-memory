---
title: 等待迴圈的 pgrep -f 會比對到自己
scope: tools/shell
status: active
updated: 2026-11-05
tags: [pgrep, ssh, watcher, background, launcher]
---

# 等待迴圈的 `pgrep -f` 會比對到自己

`until ssh host '! pgrep -f "bench3.py gpu gpu6"'; do sleep 300; done` 這類等待迴圈永遠不會結束：遠端 `bash -c` 自己的命令列就含有那串字，`pgrep -f` 一定找得到自己。2026-10-29 主 session 有 6 個這樣的 waiter 卡了好幾個小時，RTL session 的 NVDLA watcher（`pgrep -f "yosys -l nvdla3"`）在 yosys 早上 07:03 結束後白等了 13 小時，ABC watcher 也因同樣原因沒回報。

變體：除了 waiter 本身，**父層 launcher 的命令列**也會被配到。2026-11-05 agentic-ic3 的 p31 launcher 用 `bash -c '... python3 scripts/run_matrix.py ...'` 等時間到才開跑，監看程式用 `pgrep -f run_matrix.py` 判斷 run 是否在跑，結果配到 launcher 外層 shell，誤判 p31 已開始，一路等到時間上限，期間等於沒監看。字元類別 `[r]un_matrix` 也救不了這種情況，因為 launcher 命令列裡本來就有字面字串。

做法（擇一）：

- 用 `^` 錨定在真正程序的開頭：`pgrep -f '^python3 scripts/run_matrix.py'`。這可以同時排除 waiter 自己和外層 shell。

- 用字元類別讓樣式不等於字面字串：`pgrep -f "[b]ench3.py gpu gpu6"`。
- 程式名精確比對：`pgrep -x gpuls`。
- 直接等 PID：啟動時記下 `$!`，之後 `while kill -0 $pid 2>/dev/null; do sleep 60; done`（遠端則 `ssh host "while kill -0 $pid; do sleep 60; done"`）。
- 等輸出檔：完成時 `touch done.flag`，迴圈等那個檔。

waiter 超過預期時間還沒回報時，直接去看目標程序和輸出檔，不要假設它還在跑。
