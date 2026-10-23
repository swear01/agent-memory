---
title: rIC3 各版本 portfolio 組態差異：1.5.2 吃 BTOR2 時沒有 bit-level kind
scope: tools/ric3
tool: rIC3
status: active
created: 2026-10-23
updated: 2026-10-24
tags: [ric3, portfolio, btor2, aiger, k-induction, hwmcc24, benchmarking]
---

# rIC3 portfolio 組態依版本與輸入格式而不同

2026-10-22／23 在 mazu 用 PicoRV32 riscv-formal（ADD、SLTI、ORI）實測，並對照 `gipsyh/rIC3` 原始碼。

## 三個版本

- **HWMCC'24 版**：`gipsyh/rIC3-HWMCC24` repo 直接附預編 `rIC3` binary（最後一次重編是 2024-09-12，`--version` 顯示 `rIC3 1.0.0 commit_hash:28497e91`；rIC3 repo 的 tag `HWMCC24` 指向 `9b604267`）。舊 CLI 為 `rIC3 [-v N] [-e ENGINE] <model>`，預設 portfolio 有 16 個 worker：11 個 IC3、4 個 BMC、`kind --step 1`；AIGER 和 `.btor/.btor2` 用同一組 worker（BTOR2 先經 `btor_to_aiger`）。成功時 exit 20，並印出 `result: true`。
- **1.5.2**（crates.io 與 GitHub 最新 release，2025-12-07）：`src/portfolio/portfolio.toml` 依副檔名選組態。`.aig/.aag` 用 `bl_default`，裡面有 `-e kind --step 1`；`.btor/.btor2` 用 `wl_default`，**沒有 bit-level `kind`**，改成 `wl-kind --step 1` 與 `wl-bmc`。
- **master `8c6df1b`**（2026-10-15，`v1.5.2-106-g8c6df1b`，Cargo 版本仍是 1.5.2，沒有新 release）：CLI 改成 `ric3 check [--ui false] <model> portfolio`，不論輸入格式一律用 `bl_default`，含 `kind --step 1 --simple-path`。結果只印 `UNSAT`，**exit code 一律 0**，必須讀輸出判斷 verdict。可用穩定版 rustc 1.93 執行 `cargo build --release --locked` 建置（設定獨立的 `CARGO_HOME`／`CARGO_TARGET_DIR`）。

## 實測影響（同一個 `write_btor -s` 產生的 BTOR2）

riscv-formal 只在 cycle 20 檢查一次，bit-level k-induction 在 depth 21 即可完成證明。

| rIC3 | 參數 | ADD | SLTI | ORI |
| --- | --- | --- | --- | --- |
| HWMCC24 | portfolio | 26.3 s | 11.2 s | 18.9 s |
| 1.5.2 | portfolio | TIMEOUT（1800 s，先前 69 分鐘也沒有結果） | TIMEOUT | 581 s（IC3 勝出） |
| 1.5.2 | `-e kind --step 1` | 8.1 s | 5.7 s | 6.4 s |
| 1.5.2 | `-e wl-kind --step 1` | 停在 depth 20 之後，1800 s TIMEOUT | — | — |
| master | `check portfolio` | 69.8 s | 163 s | 64 s |

## master 為什麼比 HWMCC24 慢（2026-10-24 對照）

三個版本最後都是 kind 勝出，差別在 portfolio 裡的 kind 用什麼參數。master `bl_default` 的 kind 從 commit `92f17e4`（2026-01-28，CIll 相關）起多了 `--simple-path`；同一份 `portfolio.toml` 的 `[bl_hwmcc]` 仍是 `kind --step 1`，可用 `ric3 check <model> portfolio --config bl_hwmcc` 選。ADD B0、master：

| 參數 | 各次時間 |
| --- | --- |
| `kind --step 1` 單跑 | 7.39, 7.52, 7.49 s |
| `kind --step 1 --simple-path` 單跑 | 35.21, 34.93, 34.68 s |
| `portfolio`（預設 `bl_default`） | 69.77, 68.62, 100.10 s |
| `portfolio --config bl_hwmcc` | 30.45, 30.31, 17.52 s（HWMCC24 是 26.3 s） |

`--simple-path` 讓這題的 kind 慢約 4.7 倍；在 16 個 worker 的 portfolio 裡，kind 還會再慢 2～4 倍（7.5 s 對 26～30 s，35 s 對 70～100 s），推測是與其他 worker 搶核心和記憶體頻寬，沒有另外量測。

1.5.2 改吃 AIGER 時，`bl_default` 中的 kind 會勝出，ADD 約 40 s。所以要比較時間時，應改用 AIGER 或 master，或直接指定 `-e kind`。

## 其他陷阱

- **HWMCC24 版只檢查第一個 bad**（`src/transys/mod.rs` 取 `aig.bads[0]`）。拿多 property 的 AIGER 去跑時，結果只代表第一個 assert。1.5.2 與 master 則會 OR 全部 bad（`compress_bads`）。
- HWMCC24 版和 1.5.2 的 portfolio temp dir 都寫死在 `/tmp/rIC3/`，worker 與父行程在同一個 process group，可用 `killpg` 收掉；時限問題見 [portfolio 時限](portfolio-time-limit-not-enforced.md)。
- 實驗紀錄與 [NeuroAbs decision 實驗](../../projects/rtl-relate/neuroabs-decision-experiment-2026-10-22.md) 的 ADD pilot 相關。
