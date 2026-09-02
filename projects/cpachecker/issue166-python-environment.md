---
title: CPAchecker issue 166 Python classifier environment
project: swear01/cpachecker
tags: [vguide, issue166, python, z3, reproducibility]
status: active
created: 2026-09-02
updated: 2026-09-02
---

# 目前狀態

#166 的 schema-11 deterministic controls 已正式通過；issue 仍 open，下一步是另行 freeze 的
model-free semantic classifier。External model calls 維持 0，重建 Python 分析環境不等於啟動
formal run。

# 環境漂移根因

舊研究腳本沒有 requirements 或 task-local venv，直接依賴 host `python3`。系統紀錄顯示
2026-08-22 Ubuntu release upgrade 移除了 Python 3.10、`python3.10-venv` 與其 library packages，
改用 Python 3.14；因此原 interpreter-bound Python packages 不會自動跟著升級。裸
`python3` 現在還會先命中 `/opt/miniconda/bin/python3`，不可當作可重現的執行入口。

# 已固定的分析環境

Canonical spec：
`<remote-home>/cpachecker-experiments/runs/issue166_semantic_precision_census_20260901/PYTHON_ENVIRONMENT.md`。

- venv：同目錄 `.venv`；
- interpreter：`/usr/bin/python3.14`，重建時為 Python 3.14.4；
- 唯一第三方 Python dependency：`z3-solver==5.1.0.0`，import name 是 `z3`；
- `requirements.txt` 是 source of truth，`.venv` 是可丟棄的 build product；
- 呼叫時顯式用 `.venv/bin/python`，不要依賴 activation 或裸 `python3`。

相關 #166 與 reused #138 Python files 的其他 imports 全是 stdlib 或 local modules。Shell formal
harness 的 Bash/coreutils、Git、OpenSSH、`jq`、`taskset`、`timeout`、`mpstat`、JDK 21 與
CPAchecker 是 host/runtime dependencies，不放進 Python venv。

# 已驗證

- `z3.get_version_string()` = `5.1.0`；
- `uv pip list` 僅有 `z3-solver 5.1.0.0`；
- reused issue138 `test_census.py`：6 tests passed；
- issue166 `test_schema11_controls.py`：3 tests passed；
- `prefreeze.py self-test` 與 `formal_retry.py self-test` passed；
- 所有文件列出的 shell host tools、JDK 21.0.10 與 CPAchecker launcher 存在。

Durable rule：研究用 Python solver 腳本必須把 interpreter、exact dependency 與 rebuild/test
commands 記在 artifact root；系統 release upgrade 後先重建 task-local venv，不要把 host
site-packages 當成研究 provenance。
