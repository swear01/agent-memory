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
Issue `#174` 已由 PR `#176` 在 merge commit `ffc275c2` 完成。Rust 已同步 Python
production defaults，包括 identity calibration、RV32M／trace semantics、counter
cosine、fixed-scale normalization 與 large-N CLARA clustering。Python-only、
default-off research features 仍可不同。

Vichuang chunk 驗證結果：set5／set6／set7_2 的 Python、Rust partition 完全一致；
set9 pair agreement `99.9758%`、ARI `0.997295`，BA 分別為 `0.790744`／`0.791862`。
20-profile projected mean BA 為 Python `0.699376`、Rust `0.699432`，差 `0.000056`。

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

GitHub issue `#174` 已完成 Rust 對齊 Python current defaults；PR `#176` 的 CI、
nightly VI profiles、Rust checks 與 GitHub review 均通過後合併。

Issue `#174` 刻意不要求一次移植所有 default-zero research features；Drain、char
ngram、完整 selective LLM refine／merge 等需要各自的 evidence-backed promotion。

# 驗證要求

不得只用小型 synthetic dataset 判定 parity；必須涵蓋 Vichuang chunk profiles，
包含 N=1000 CLARA 與 skewed counter histograms。比較 partition、BA、pair agreement／
ARI，以及 feature／distance numerical delta；production-default 變更須同步更新
Rust、Python、active docs 與 parity gate。
