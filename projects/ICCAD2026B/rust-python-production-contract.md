---
title: ICCAD2026B Rust/Python production contract
project: ICCAD2026B
tags: [rust, python, submission, parity, clustering, documentation]
status: verified
created: 2026-08-26
updated: 2026-08-26
---

# 現況

Rust 是唯一正式 submission runtime；Python `src/` 是 reference／research 路徑。
截至 `main` merge commit `1f21af4f`（PR `#172`），Rust 不是 Python current default
的等價實作，而是落後一個主要演算法版本：Python large-N CLARA 已用 bounded fused
structural distance 計算 `sample × sample` 與分塊 `N × medoids`，Rust large-N CLARA
仍只用 feature-vector cosine。

Source-level 稽核的設定面差距為：Python/Rust feature flags `64/19`、group scales
`37/15`、distance parameters `57/10`。這些數字包含 default-off research features，
不能解讀成 BA 百分比；真正的 production blocker 是 default-active 語意、權重與
large-N distance path 已分叉。

# 長期規則

- Production/default 行為變更必須在同一個 change／PR 同步 Rust submission；
  Python-only 實作不算完成。
- 同一個 change 必須同步 active architecture、configuration、plan、submission、
  testing 文件。
- 改變的 default 行為必須加入 Rust/Python parity test，使後續 drift 在 CI 失敗。
- Python-only research 可以存在，但必須明確標成 experimental、default-off，且不得
  宣稱為 production 行為。
- Rust implementation、active docs 或 parity coverage 缺一時，production PR 不應
  合併。

# 追蹤與驗收

GitHub issue `#174` 追蹤 Rust 對齊 Python current defaults。範圍包含：同步有效
defaults、default-relevant parser/feature semantics、Rust rectangular fused-distance
API、bounded CLARA、soft-K／K upper bound／strict capacity behavior，以及新的
Rust/Python default-parity CI gate。

Issue `#174` 刻意不要求一次移植所有 default-zero research features；Drain、char
ngram、完整 selective LLM refine／merge 等需要各自的 evidence-backed promotion。

# 容易誤判的驗證

`scripts/verify_submission.sh` 只 gate Rust binary 的 ABI、determinism、Case coverage
與 public BA floor，明確不要求與 `python -m src` partition isomorphism。
`scripts/verify_native_equivalence.py` 比較的是 Python native trace extraction 與舊
Python cache，也不是 Rust/Python parity。兩者都不能證明 Rust 已追上 Python。
