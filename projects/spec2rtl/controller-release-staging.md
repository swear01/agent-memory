---
title: Spec2RTL CI controller release 部署到 Mazu
scope: projects/spec2rtl
machine: mazu
status: active
updated: 2026-10-20
---

# Controller release 部署

`scripts/ci` 任何檔案（含 README、tests）改動都會改變 controller fingerprint；`cursor-sdk-smoke.yml` 的 `SPEC2RTL_CI_CONTROLLER_SHA256` 必須同 PR 更新，且 Mazu 上必須先有 `/opt/spec2rtl-ci/releases/<fingerprint>/`，否則 Regression 在 `Select pinned controller` 以 `Required controller release is not deployed: <sha>; refusing model calls` 失敗。這種失敗與程式正確性無關。

Fingerprint：對 `scripts/ci` 下排序後的相對路徑與檔案內容做 sha256，忽略 `__pycache__`（同 workflow selector step）。

## 步驟

1. 本機驗證：PR head 合併最新 develop 後跑 `tests/test_controller_contract.py`、`scripts/ci` 的 `test_*.py`（需 `cursor-sdk==1.0.36`）、`sdk.py --self-test`、`check_repository.py --self-test`，並重算 fingerprint 對 pin。
2. 確認 grading 相容：`grade.py`、`timing.py`、`optimize.py`、`ppa.py`、`monitors/` 未改則 `grading_sha256` 不變，既有 reference 可沿用。
3. 以 `swear01` 將 `git archive <rev> scripts/ci tests/test_controller_contract.py .github/workflows/cursor-sdk-smoke.yml` 傳到 `<remote-home>/spec2rtl-release/`，連同 staging script。
4. 以 `ssh -t su_mazu 'sudo sh <remote-home>/spec2rtl-release/<script>'` 部署（`swear01` 無 sudo，見 [管理帳號與 sudo 驗證](../../machines/hapi-fleet/system-install-via-admin.md)）。Script 用 `umask 022`、`tar --no-same-owner --no-same-permissions` 解到 tmp 目錄，寫 `release.json`（`source_revision`、`controller_sha256`、`grading_sha256`），驗 fingerprint，`chown -R root:root` 後 `mv` 到正式路徑；已存在則跳過。
5. 以 `swear01` 複驗 fingerprint、無 group/other 可寫檔、全為 root 擁有，再 `gh run rerun <run-id> --failed`，確認 `Select pinned controller` 與 `Verify controller` 通過。

保留舊 release 直到使用它的 job 結束。

## PPA handoff 的比較目錄

- 報 `Required synthesis/STA failed or is stale for the current RTL/contract` 時，核 `ppa/comparison/result.json` 的 source hashes 是否等於最終 `rtl/`、contract/library hash、報告及 symlink。revert timing candidate 常留下未採用版本的報告。
- 用同一 driver 對每組 candidate 只換 `--rtl`、`--output output/<DESIGN>/ppa/candidate-<n>/`。若採用版本與 candidate 的檔名和內容完全相同，保留原 comparison 為編號 sibling，再將 matching candidate 搬成 `comparison/`。其他情況用同指令重跑。報告以相對路徑讀取，CI 合成按 RTL hash 做 cache。
- 推修正後查 develop 實際納入最新 head。單次 Regression 要下載 evidence 才知道是否觸發 candidate 路徑。舊 CI PR 需比較 develop code 和 run 證據，non-ancestor 本身不能證明尚缺功能。

[原始紀錄](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/projects/spec2rtl/controller-release-staging.md)
