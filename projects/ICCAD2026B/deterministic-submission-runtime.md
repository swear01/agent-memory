---
title: ICCAD2026B submission 已移除官方 LLM 通道
project: ICCAD2026B
tags: [submission, llm, deterministic, clustering]
status: verified
created: 2026-08-25
updated: 2026-08-30
---

# 目前狀態

PR `#178`（merge commit `210921a0`）已在 `main` 恢復並完整落實這項決策；
`stable-2.3` 是第一個依此 contract 實際發布、上傳 GitHub Release 與 Google Drive
並獨立核對的正式版本。先前「PR `#172` 又帶回 LLM」的 superseded 狀態只描述
`stable-2.3` 之前的短暫歷史，已不是目前真相。

現在 Rust submission 與 Python reference 都沒有 submission LLM channel。Rust
manifest 不含 `reqwest`／`tokio`，`rust/src/` 不存在 `*llm*` 模組；Python `src/`
也不再包含 `llm*.py`、model config 或 `LLM_MODEL_CONFIG` production 呼叫鏈。
`tests/test_ci_contract.py` 同時鎖定 Rust、Python 與 locked Cargo build，避免只刪
其中一側後再次漂移。

# 決策

PR `#170` 首先把 submission runtime 改成完全 local、deterministic 的單一路徑；
該 PR 當時合進 `develop` 而非 `main`，所以不能只用其 merge 狀態證明正式版本已
採用。PR `#178` 將等價變更移植到 `main` 並由正式 release 驗證。
`src/__main__.py` 不再讀取官方 model config、
呼叫 endpoint、產生 embedding，或執行 completion/refinement/routing。

實作直接刪除 `src/llm*.py`、`src/llm_config/`、API key 範例、live/routing
scripts、專用 tests，以及 submission 的 `httpx`、`openai`、`python-dotenv`
依賴；沒有保留 feature flag 或 silent fallback。`tests/test_ci_contract.py` 的
`test_submission_has_no_llm_channel` 鎖定 source、requirements 與 build script
都不得重新引入該通道。

# 邊界

`scripts/pr_gen/ibex_pr_bug_miner` 是獨立的離線 PR mining 研究工具，仍可使用
OpenAI，但不被 submission runtime、submission requirements 或 build 引入。
這不是官方 evaluator LLM 通道的一部分。

# 容易誤刪的 helper

`src.__main__._runtime_budget` 即使不再由正式 runtime 呼叫，仍被
`scripts/evaluate_k_selection.py` 匯入。移除 LLM 呼叫鏈時曾把它判成 dead code，
完整 fast suite 隨即抓到離線 evaluator import regression；因此不能只看 module
內 call site。刪除看似未使用的 submission helper 前，要先跨 `scripts/` 搜尋 import。

# 驗證基準

PR `#178` 最新 head 通過 1,332 個 fast tests、3 個 nightly tests、Rust unit／
integration、clippy、fmt、Rust/Python parity，以及 GitHub 的 Python 3.13/3.14、
nightly、baseline、secret scan 與 review checks。正式 release 另通過 RHEL 8／
GLIBC 2.28 build、cold-cache build、公開 sanity gates 與 Drive publication。
