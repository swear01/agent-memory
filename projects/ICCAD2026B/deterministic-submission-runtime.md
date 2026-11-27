---
title: ICCAD2026B submission 已移除官方 LLM 通道
scope: projects/ICCAD2026B
project: ICCAD2026B
tags: [submission, llm, deterministic, clustering]
status: verified
created: 2026-08-25
updated: 2026-11-28
---

- 正式 ICCAD2026B submission 用 deterministic local Rust，Python reference 同契約。Submission 不讀官方 model config、不呼叫 endpoint、embedding 或 completion 構件，也不留 feature flag／fallback。Rust manifest 不帶 `reqwest`／`tokio` 或 `*llm*` module，Python runtime 不帶 `llm*.py`、`LLM_MODEL_CONFIG` 或網路模型依賴。
- `scripts/pr_gen/ibex_pr_bug_miner` 是離線 PR 挖掘研究工具，可用 OpenAI，保持不被 submission runtime、requirements、build 匯入。
- 從 submission 刪 helper 前跨 `scripts/` 搜尋 imports；`src.__main__._runtime_budget` 仍被 `scripts/evaluate_k_selection.py` 使用。
- 跑 `tests/test_ci_contract.py` 的 `test_submission_has_no_llm_channel` 及 locked Cargo build、Rust/Python parity、完整 fast suite。查目標 release 的主線、實際包與 Drive，不用 develop merge 當發布證據。首次已驗 release 為 stable-2.3。

[原始紀錄](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/projects/ICCAD2026B/deterministic-submission-runtime.md)
