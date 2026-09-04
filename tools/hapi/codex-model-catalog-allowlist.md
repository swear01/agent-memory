---
title: HAPI Codex model restriction depends on an explicit local catalog
scope: tools/hapi
project: hapi
tool: Codex app-server
status: active
confidence: high
created: 2026-09-03
updated: 2026-09-04
tags:
  - hapi
  - codex
  - model-catalog
  - allowlist
source_refs:
  - hapi:canonical-gist
  - local:<project-root>/hapi-handover.md
  - live:zeus-2026-09-03
  - live:swop-2026-09-04
  - github:swear01/transfer_MAC#38
generated_by: openai-codex/gpt-5.6-sol
---

# Configuration boundary

HAPI 的新 session 模型選單顯示 Codex app-server `model/list` 的有效可見 catalog。`model = "..."` 只指定預設模型，不會限制選單；限制必須由 `~/.codex/config.toml` 的絕對 `model_catalog_json` 路徑與實際存在的 `~/.codex/models.allowlist.json` 一起成立。

設定行或檔案缺失、無效時，Codex 會 fail open 到完整內建 catalog，而不是回報 allowlist 錯誤。因此只看預設模型或設定檔存在不足以判定 HAPI 選單受限。

# Fleet policy

每個獨立 home 都要保留 canonical allowlist，並讓 `model_catalog_json` 指向該 host 的絕對路徑。更新 catalog 後先比較各 host 檔案內容，再實際呼叫 app-server `model/list` 或 HAPI machine Codex-models API，比對可見 ID。

2026-09-03 驗證的可見集合為：

- `gpt-5.6-sol`
- `gpt-5.6-terra`
- `gpt-daybreak-blue-latest`
- `gpt-5.6-luna`

不要用 allowlist JSON 的 entry 數量代替有效驗證；canonical 檔案也可包含不會出現在一般選單的 hidden/internal models。

# Documentation mirrors

Durable HAPI changes must keep the canonical Gist `hapi-readme.md`, `<project-root>/hapi-handover.md`, and the Skillshare `references/gist-latest.md` plus `references/hapi-readme.md` byte-identical. QMD indexes this shared-memory pointer, not the full playground handover.

# Zeus incident

Zeus 曾同時缺少 `model_catalog_json` 與 allowlist 檔案，`model/list` 因而額外顯示 `gpt-5.5`、`gpt-5.4`、`gpt-5.4-mini`、`gpt-5.3-codex-spark`。保留 `config.toml` 備份、補回 canonical 檔案與絕對設定行後，不需重啟 HAPI runner，新的 `model/list` 即恢復四個可見模型；runner PID 與 restart count 未變。

# Windows restore incident

2026-09-04，`swop` 也因為本機缺少 `model_catalog_json` 與 `models.allowlist.json` 而顯示舊模型。根因不在 updater policy：updater 已存在於 shared-skills shelf，但 `transfer_MAC` 的 canonical `sync-ai-agent-configs.py render` 沒有安裝或呼叫它，Windows restore 因而漏掉手動步驟。

`transfer_MAC` PR #38（merge commit `a1b63ec`）把這個缺口納入 canonical renderer：完整 `render` 與 Windows 專用 `render-codex-models` 都會安裝並執行 updater、驗證 catalog root／model entries／visible slugs、保留既有 TOML newline，再寫入絕對 `model_catalog_json`。缺少 skills submodule、updater 執行失敗或 catalog 無效時必須 fail closed；不要把 model 更新改成 optional，否則會重現本次 silent fail-open。

Live app-server `model/list` 驗證只剩四個可見模型，`gpt-5.5` 與 `gpt-5.4*` 的可見數量為零。這類修復要同時更新目前 host 與 canonical restore path；只修 live file 仍會在下一次重建時復發。
