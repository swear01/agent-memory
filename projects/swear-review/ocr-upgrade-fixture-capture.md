---
title: Swear Review OCR 升級時的 contract fixture 擷取方法與陷阱
scope: projects/swear-review
project: swear-review
status: active
created: 2026-10-20
updated: 2026-11-20
tags: [swear-review, ocr, open-code-review, fixtures, upgrade, gateway]
source_refs:
  - https://github.com/swear01/swear-review/pull/11
---

- **升 OCR 後啟動報 `tools_file` validation 或假裝 YAML 已切版本。** 1.13.8 移除 `--tools`，目前 adapter 拒絕舊工具檔欄位，部署前移除 prod `ocr.tools_file`。Oracle 的 `ocr.version` 只是標籤；核實 PATH 的 `/usr/local/bin/ocr` 及服務 startup log。1.13.13 process native 名稱 `opencodereview`，CLI 還是 `ocr`。Unknown JSON 如 groups/retry_report/thinking 不自行當 contract break；驗 status、coverage、classification、run_failure。
- **抓 fixture 看到 404 或 MissingSessionID。** 用 `npm install --prefix <scratch> @alibaba-group/open-code-review@<ver>`，platform binary 由 optionalDependencies 裝。沿 runner 的隔離 HOME、`OCR_NO_UPDATE=1`、`--concurrency 1 --format json --audience agent --timeout 15` 和本機 gateway；`OCR_USE_ANTHROPIC=false` 避免把 `/v1/messages` 接到 OpenAI 路徑。本機 gateway 依現行設定帶 session header；清理 fixture comments／code／thinking 及私人資訊。
- **Cancelled 沒有 JSON、partial 變成全 failed。** 只向 native child 送一次 SIGTERM，group SIGTERM 可能由 shim 再送一次而丟 stdout。Partial 用三個可拆 group 的獨立目錄；1.13.13 的 50K budget 在 dispatch 前全失敗，120K 可作 fixture 起點。Failed 用本機 500 stub、skipped 用純 README 改動。1.13.12 起 complete／skipped／budget-only partial exit 0，failed／quota／cancelled exit 1；先解析 stdout，不能從 RC 單獨判定所有 groups。
- **服務看似 idle 或 health 綠就報完成。** 在 Oracle 用唯讀 DB 確認 `review_jobs`／`review_runs` 無 queued/running，檢查 OCR 程序，累計 metrics 不能代替。保留 OCR 全域目錄、app dist、config 和適當 DB 備份，再升級 build/runtime，核 startup 及 health/ready。用 open PR 的真實 full review 核版本、terminal coverage 和結果；已合併 PR 上指令不建立 job。Rollback 同時復原 binary、app revision/dist 與 config，重啟後再驗。

[原始紀錄](https://github.com/swear01/agent-memory/blob/9b2d556e370f3a71736d5dc0a63f3b1e1c8c3a0e/projects/swear-review/ocr-upgrade-fixture-capture.md)
