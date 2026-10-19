---
title: Spec2RTL CI controller release 部署到 Mazu
scope: projects/spec2rtl
machine: mazu
status: active
updated: 2026-10-19
---

# Controller release 部署

`scripts/ci` 任何檔案（含 README、tests）改動都會改變 controller fingerprint；`cursor-sdk-smoke.yml` 的 `SPEC2RTL_CI_CONTROLLER_SHA256` 必須同 PR 更新，且 Mazu 上必須先有 `/opt/spec2rtl-ci/releases/<fingerprint>/`，否則 Regression 在 `Select pinned controller` 以 `Required controller release is not deployed: <sha>; refusing model calls` 失敗。這種失敗與程式正確性無關。

Fingerprint：對 `scripts/ci` 下排序後的相對路徑與檔案內容做 sha256，忽略 `__pycache__`（同 workflow selector step）。

## 步驟

1. 本機驗證：PR head 合併最新 develop 後跑 `tests/test_controller_contract.py`、`scripts/ci` 的 `test_*.py`（需 `cursor-sdk==1.0.36`）、`sdk.py --self-test`、`check_repository.py --self-test`，並重算 fingerprint 對 pin。
2. 確認 grading 相容：`grade.py`、`timing.py`、`optimize.py`、`ppa.py`、`monitors/` 未改則 `grading_sha256` 不變，既有 reference 可沿用。
3. 以 `swear01` 將 `git archive <rev> scripts/ci tests/test_controller_contract.py .github/workflows/cursor-sdk-smoke.yml` 傳到 `<remote-home>/spec2rtl-release/`，連同 staging script。
4. 以 `ssh -t su_mazu 'sudo sh <remote-home>/spec2rtl-release/<script>'` 部署（`swear01` 無 sudo，見 [[machines/mazu/admin-access]]）。Script 用 `umask 022`、`tar --no-same-owner --no-same-permissions` 解到 tmp 目錄，寫 `release.json`（`source_revision`、`controller_sha256`、`grading_sha256`），驗 fingerprint，`chown -R root:root` 後 `mv` 到正式路徑；已存在則跳過。
5. 以 `swear01` 複驗 fingerprint、無 group/other 可寫檔、全為 root 擁有，再 `gh run rerun <run-id> --failed`，確認 `Select pinned controller` 與 `Verify controller` 通過。

保留舊 release 直到使用它的 job 結束。

## 2026-10-19 紀錄

同學將 orchestrator 改為讀 `flow.toml`（PR #74、#76）。PR #75 讓 CI 在 workspace 寫入唯讀的 `flow.toml`（`clarify-spec = "auto"`、`alignment-check = "full"`、approve/escalation 維持 manual）與 `eda_env.sh`，pin `33a447ee…`。PR #77 疊在 #75 上，因 `skills/refactor-style` 將 `parallel-generation.md`／`parallel-verification.md` 合併為 `parallel-workers.md`，`sdk.py` 改以 `parallel_policy()` 同時接受新舊檔名，pin `7e148b63…`。兩個 release 皆已部署，重跑的 Regression 均通過 controller 選擇與驗證。
