---
title: Pi fleet 的 DeepSeek model ID 遷移與四層 allowlist
scope: tools/pi
project: hapi
tool: pi
status: active
confidence: high
created: 2026-09-11
updated: 2026-09-12
tags: [pi, deepseek, gateway, model-filter, fleet, swop]
---

## Pi 的有效模型清單

改模型 ID 時，只換指定模型。保留所有其他模型、provider、default provider/model/thinking 與非互動行為。2026-09-11 曾順便把 allowlist 改到只剩兩個 DeepSeek ID，HAPI picker 因此遺失 Codex、Meta 與 Qwen，次日從改名前的各機備份復原。

- `~/.pi/agent/settings.json` 的 `enabledModels` 控制 Pi 預設與循環選擇，不能用來判斷完整 catalog。
- `~/.pi/agent/model-filter.json` 的 `defaultAction: block` 需要安裝 `pi-model-filter` 才有效。保留既有 provider 規則。
- `~/.pi/agent/extensions/strict-model-allowlist.ts` 在常駐 session 內限制 API 呼叫。除了 `allowed` Set，Meta 的放行還涉及 `includes` 及 `model-filter.json` 條目。
- `~/.local/bin/pi` 可能是 `pi-safe` wrapper 或 shell alias，有獨立的 `awk` allowlist。Pi 0.85.x 的 `pi --list-models` 直接讀 `ModelRuntime`，不會跑 extension 的 `session_start`。CLI、HAPI RPC picker 與 ACP 是不同消費入口，必須逐一讀回。
- 新 ID 必須出現在 `models.json` 的 `providers["opencode-go"].models`。內建 provider 的舊 catalog 可能來自 `models-store.json`，光改 `enabledModels` 不會建立新模型。

先用 `command -v pi` 與 symlink／alias 檢查找出實際入口，再在 session 讀 `ctx.modelRegistry.getAll()/getAvailable()`，用 `GET /api/machines/:id/pi-models` 驗證 HAPI picker。發現 `RPC handler not registered` 時先查 runner 註冊狀態。2026-09-12 用這個 API 已讀回六台 Linux 與 Mac 的非 DeepSeek 模型與新 ID，當時未驗證 Windows swop。

## Gateway、舊 session 與復原

2026-09 改名將 Pi ID 換成 `opencode-go/deepseek-flash`、`opencode-go/deepseek-pro`。保留 gateway 的舊別名，讓既有 client 可以遷移。CLI 設定仍需依實際 endpoint 的 model ID 配置，尤其不要把本機別名套到官方或 OpenRouter endpoint。

- 在 `config.yaml` 更新 `models.allow`，在 `routing.yaml` 為每個 upstream 設實際可用的 `upstream_model`，例如當時 Command Code 的 `deepseek/deepseek-v4.1-flash` 與 `deepseek/deepseek-v4-pro`。舊 Bun gateway 的 route 和 failback 見 [封存操作筆記](../../archive/projects/deepseek-latch-gateway/fleet-routing-and-failback.txt)。
- 共用 NFS 設定檔只改一次，gateway 在程序啟動時讀取設定，所以每台都要重啟及驗證 `POST /v1/chat/completions` 的回應 `model` 與實際 upstream。單看 `/health` 無法確認路由。
- Pi 會以 `Model "..." is blocked by strict-model-allowlist` 拒絕新啟動時明確指定的舊 ID。已啟動的 session 持有 model 物件，續跑前不用換。重新 resume 如果無法 resolve 舊 ID，可能選用 `currentModel`、預設或第一個可用模型。
- 讀回備份的 mtime、`enabledModels` 與 filter rules，再決定是否還原。當時 `~/.config/deepseek-gateway/backup-<timestamp>-pre-deepseek-rename/` 的部分 `agent__*.json` 雖然名稱寫 pre-rename，實際已含被砍成兩筆的 allowlist。Mac 上確認有效的副本是 Pi 原檔旁的 `.backup-deepseek-rename-20260911-113603`，還原時仍須逐機核對。

## Windows swop 待驗

使用者要求保留待辦。重試前先查 runner／SSH 可達性，並對照當下 gateway 與 upstream 設定。Pi 位於 Windows 使用者 home，gateway 由 SYSTEM Scheduled Task `DeepSeek Gateway (SWOP)` 管理，設定在 `<program-data>/DeepSeekGateway`。若 runner 仍未註冊，須在 Windows 本機核對 registry，並測兩個 gateway model 的實際請求，不宜以 hub 讀取失敗推斷設定。

部署明細、當時的 Windows 步驟、備份與驗證可由 `git show 54da0f0adfdb3dacb3bca59a729b12975d44ec9b:tools/pi/deepseek-model-id-migration.md` 追溯。設定來源修正見 transfer_MAC PR #46、#47。
