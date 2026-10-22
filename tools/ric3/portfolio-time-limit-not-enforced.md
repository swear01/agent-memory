---
title: rIC3 1.5.2 portfolio 不遵守 --time-limit
scope: tools/ric3
tool: rIC3
status: active
created: 2026-10-22
updated: 2026-10-22
tags: [ric3, portfolio, timeout, benchmarking]
---

# rIC3 1.5.2 portfolio 不遵守 `--time-limit`

2026-10-22 在 mazu 實測（rIC3 1.5.2，crates.io 當時最新版，SHA-256 `f57d2501…182c`）：`rIC3 --engine portfolio --time-limit 3600 model.btor2` 在 PicoRV32 riscv-formal ADD 模型上跑了 69 分鐘仍未結束；`--time-limit 5` 同樣被忽略。portfolio 會啟動十多個 worker process，只殺父行程不會停掉 worker。

做法：用 `subprocess.Popen(..., start_new_session=True)` 啟動，`wait(timeout=cap+grace)` 逾時後 `os.killpg(pid, SIGKILL)`，結果記為 `TIMEOUT`（不是 proof）。NeuroAbs 的 `src/run_decision_experiment.py` 已這樣實作（cap+60 秒）。若用 agent harness 的背景工作監看，harness 自己的時限也可能先砍掉工作而得不到 verdict；長跑應 `nohup setsid` 脫離。

其他觀察：portfolio 中的 `bmc --bmc-kissat` worker 常在 PicoRV32 上 `kissat: fatal error: out-of-memory reallocating from 4294967296 to 8589934592 bytes`；其他引擎照常繼續，不影響 verdict 判讀。退出碼 UNSAT 20、SAT 10、UNKNOWN 30，需與印出的 verdict 一起檢查。相關實驗見 [NeuroAbs decision 實驗](../../projects/rtl-relate/neuroabs-decision-experiment-2026-10-22.md)。
