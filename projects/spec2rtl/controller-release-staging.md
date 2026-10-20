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
4. 以 `ssh -t su_mazu 'sudo sh <remote-home>/spec2rtl-release/<script>'` 部署（`swear01` 無 sudo，見 [[machines/mazu/admin-access]]）。Script 用 `umask 022`、`tar --no-same-owner --no-same-permissions` 解到 tmp 目錄，寫 `release.json`（`source_revision`、`controller_sha256`、`grading_sha256`），驗 fingerprint，`chown -R root:root` 後 `mv` 到正式路徑；已存在則跳過。
5. 以 `swear01` 複驗 fingerprint、無 group/other 可寫檔、全為 root 擁有，再 `gh run rerun <run-id> --failed`，確認 `Select pinned controller` 與 `Verify controller` 通過。

保留舊 release 直到使用它的 job 結束。

## 2026-10-19 紀錄

同學將 orchestrator 改為讀 `flow.toml`（PR #74、#76）。PR #75 讓 CI 在 workspace 寫入唯讀的 `flow.toml`（`clarify-spec = "auto"`、`alignment-check = "full"`、approve/escalation 維持 manual）與 `eda_env.sh`，pin `33a447ee…`。PR #77 疊在 #75 上，因 `skills/refactor-style` 將 `parallel-generation.md`／`parallel-verification.md` 合併為 `parallel-workers.md`，`sdk.py` 改以 `parallel_policy()` 同時接受新舊檔名，pin `7e148b63…`。兩個 release 皆已部署。#75、#77 由 wei93111 於 2026-10-19 合併進 develop（merge `f3725fc`），develop pin 為 `7e148b63…`。

#77 重跑的 run 39251538526（含 #75 全部變更）走完整流程，117.38 分鐘 FAIL：bch 通過（獨立 6/6、STA passed、AT −31.96%）；keccak 以 `Required synthesis/STA failed or is stale for the current RTL/contract` 失敗。原因不是 CI 變更：postsynthesis-ppa 試了 timing candidate 後 revert，保留原 RTL，但 `ppa/comparison/` 留著落選 candidate 的結果，保留版本的結果放在 `comparison-1/`；`sdk.py` 的 `validate_timing_handoff()` 只看 `comparison/result.json` 的 `source_hashes` 是否等於最終 RTL，因此判 stale。skill 的 revert 交接原本沒有要求 `comparison/` 必須對應被保留的 RTL。

修正（使用者授權）：PR #81 與 #82（2026-10-20 合併，develop `ec6f84c`）在 `postsynthesis-ppa/references/contract-and-sta.md` 規定 candidate 一律以同一指令只換 `--rtl` 與 `--output output/<DESIGN>/ppa/candidate-<n>/` 測量；交接時 `comparison/` 必須是 source hashes 等於 `rtl/` 的 driver run。交出的 RTL 與某 candidate 檔名與內容完全相同時，先把現有 `comparison/` 保留為編號 sibling，再把該 `candidate-<n>/` 搬成 `comparison/`；否則用同一指令重跑。搬移有效，因為 `parse_synthesis()` 以目錄相對路徑讀報告，`validate_timing_handoff()` 只比對報告、`source_hashes`、contract/library hash 與 symlink；CI 合成服務以 RTL hash 快取，重跑同一 RTL 也只回傳快取。

注意：wei93111 可能在 PR 後續 commit 推上前就合併；推完修正後要確認 develop 真的包含最新 head（#81 合併於 `6bf8e20`，後兩個 commit 需另開 #82）。每次 CI 每個 design 只跑一次，單次 Regression 不足以驗證只在特定模型行為下才觸發的 skill 規則。

#82 head 的 run 39365725844（101.59 分鐘）整體 `warning`、job 通過：keccak 與 bch 功能與 STA 均通過；AT 相對固定參照 keccak +265.72%（退步，超過 5% 只警告）、bch −40.16%。未下載 evidence 確認該 run 是否實際走到 candidate 路徑。

舊 PR #64（`ci/oracle-reference-resolution`，2026-10-15）於 2026-10-20 關閉、未合併：固定參照 run 38370030437、AT 計分與有界 PPA 優化已在 develop；逐 run 選 controller 的步驟已被 #70 的固定 release 取代。分支 commit 雖非 develop 祖先，但合併只會回退較新的 CI 程式。之後判斷舊 CI PR 是否還需合併時，先比對 develop 的 `scripts/ci` 內容與現行 run 報告，而非只看 commit ancestry。
