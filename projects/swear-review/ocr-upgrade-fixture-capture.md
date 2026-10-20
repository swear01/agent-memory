---
title: Swear Review OCR 升級時的 contract fixture 擷取方法與陷阱
scope: projects/swear-review
project: swear-review
status: active
created: 2026-10-20
updated: 2026-10-20
tags: [swear-review, ocr, open-code-review, fixtures, upgrade, gateway]
source_refs:
  - https://github.com/swear01/swear-review/pull/11
---

# 結果

`swear01/swear-review` PR #11 將 OCR 從 1.12.9 升到 1.13.2，merge commit
`927de3c4ef0aa47273b2e6ee253182193b81f35c`。新增
`tests/fixtures/ocr-v1.13.2-{complete,failed,partial,cancelled,skipped}.json`。
1.13.2 多了頂層 `groups`、`retry_report` 與 comment 的 `thinking`；adapter 的 zod
schema 會忽略未知欄位，所以不需改 adapter。status、`manifest.coverage`、
failure classification 與 `run_failure` 形狀不變。上游
`internal/config/toolsconfig/tools.json` 在 v1.12.9 與 v1.13.2 的 blob 相同
（`1b3275f`），`config/ocr-tools-no-search.json` 不需重產。
合併當下 **Oracle 尚未部署**，仍跑 1.12.9（見下方部署步驟）。

# 擷取方法（Mac，本機 gateway）

- 在 scratch 目錄 `npm install --prefix <dir> @alibaba-group/open-code-review@<ver>`；
  npm 的 allow-scripts 會擋 postinstall，但平台 binary 走 optionalDependencies，
  `ocr version` 仍可用。
- 照 `src/review/ocr-runner.ts` 的參數跑：隔離 `HOME`、`OCR_NO_UPDATE=1`、
  `--concurrency 1 --format json --audience agent --timeout 15 --repo <probe>`。
  LLM 指向本機 gateway `http://127.0.0.1:35001/v1/chat/completions`、
  model `deepseek-flash`，token 隨便填即可。
- **必須設 `OCR_USE_ANTHROPIC=false`**（worker 由 `llm.use_anthropic` 傳入）。
  漏設時 OCR 走 Anthropic 協定，路徑變成 `.../v1/chat/completions/v1/messages`，
  gateway 回 404，結果看起來像 provider failed，容易誤判。
- failed：把 URL 指到回 500 的本機 server；skipped：只改 README 的 commit。
- partial：小改動會被併成單一 `small change set` group，budget 無法造成 partial。
  需三個不同目錄、各約 60 個函式的檔案，OCR 才拆成三個 group；
  `--max-tokens-budget 25000` 太小會在任何 dispatch 前全數 failed(budget)，
  `50000` 得到 selected 3／completed 2／failed 1 (budget)。
- cancelled：只對 native `ocr-darwin-arm64` 程序送一次 SIGTERM，會輸出
  `run_failure.classification=cancelled` 的 JSON。對整個 process group 送 SIGTERM
  時，node shim 會再轉送一次，native 收到兩次訊號，stdout 為空。Swear 的 worker
  在 `proc.killed` 時直接走 cancel 路徑、不解析 stdout，所以正式環境不受影響，
  但 fixture 不能用 group kill 擷取。
- 清理：comment 的 `content`／`existing_code`／`suggestion_code`／`thinking`
  換成 sanitized 文字；確認沒有本機路徑或 email 外洩。

# Oracle 部署現況（2026-10-20 唯讀查核）

systemd unit `swear-review`、`/opt/swear-review/app` 在 `1cf91e0`；全域
`/usr/local/bin/ocr` 為 root 擁有的 npm global（v1.12.9）；
`/opt/swear-review/data/config.yaml` 的 `ocr.version: "1.12.9"`、
`concurrency: 4`、`tools_file` 指向 bundled no-search 工具檔。
升級需：確認 review queue 空 → `npm install -g @alibaba-group/open-code-review@1.13.2`
（需 root）→ app 切到新 merge commit、`npm ci && npm run build` → config 的
`ocr.version` 改為 `"1.13.2"` → restart → `/healthz`、`/readyz`，再以一個新的
PR review 確認 summary 顯示 `OCR: v1.13.2` 且 coverage 無失敗。
