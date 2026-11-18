---
title: 用 HAPI 派多個模型做程式審查
scope: tools/hapi
tool: hapi spawn-peer
status: active
updated: 2026-11-18
tags: [hapi, review, codex, claude, agy, gemini, cursor, grok, dsh, permission-mode]
---

# 用 HAPI 派多個模型做程式審查

2026-10-29 在 CULS-fast PR 上用 HAPI 派 Codex、Claude、Gemini（agy）、OpenCode、Cursor、Grok、DeepSeek（dsh）做靜態審查。

## 權限模式：明確指定，Claude 審查用 Auto

2026-10-29 使用者曾要求無人看管的審查開高權限。2026-11-06 再次明確要求 Claude Code 新 session 開 Auto，因此 Claude 審查也依 [[session-permission-preference]] 明確傳 `--permission-mode auto`；這取代先前 Claude 的高權限例外。多個 session 可開在同一資料夾。

`--permission-mode` 合法值（取自 `hapi` binary 字串）：

| agent | 高權限值 | 備註 |
|---|---|---|
| claude | `auto` | 使用者 2026-11-06 確認；省略時是 `default`（Manual） |
| codex | `yolo` | `read-only` 連 FINDINGS.md 都寫不了 |
| agy（Gemini） | `always-proceed` | 另一個是 `request-review` |
| cursor | `yolo` | |
| grok | `bypassPermissions` | 不接受 `yolo`（bad_args） |
| opencode | `yolo` | |
| dsh（DeepSeek） | 無此旗標 | |

教訓：使用者提高權限要求後，只改了新開的 session，先前用 `read-only`（Codex）和 `default`（Claude）開的兩個繼續跑了幾小時、零產出，使用者很不滿。要求一改，就檢查所有執行中的 session（`spawn_*.json` 的 `session.permissionMode` 或 `hapi inspect-peer`），模式不對先用支援的 `hapi set-mode <exact-session-id> <mode>` 修正；不支援時才按 session-control 的清理與重開流程處理。只操作本次自己建立或使用者精確指定的 session。開完立刻確認 `permissionMode`。

## 登入狀態（2026-10-29）

- agy 只在 Mac（swairM5）登入；mazu、cthulhu、zeus 沒有。
- cursor-agent：2026-11-17 起各機已重新登入並驗證（舊 token 失效時 spawn 會回 HTTP 502；重登見 cursor-login-nfs-shared-device-flow）。
- grok 只裝在 Mac 且未登入（`grok login`）。
- dsh 在 mazu 沒有 `DEEPSEEK_API_KEY`（`dsh --profile headless` 回 MISSING_CREDENTIAL）。使用者 2026-10-30 決定不用 dsh；DeepSeek 改用 OpenCode：`--agent opencode --model deepseek/deepseek-v4-pro --permission-mode yolo`（Mac 與 mazu 的 opencode 都已綁好 DeepSeek）。
- CLI 沒登入時 HAPI session 會無聲結束（`session_ended`）；先用 headless 一行指令測試。

## 操作細節

HAPI 0.35 的做法（指令對照見 [[peer-cli-0-35-command-changes]]）：

- 在 session 裡用 MCP `spawn_peer`（明確帶 `agent`、`permissionMode`），`waitForReply: true` 後結束這一輪，回覆會以新訊息送回。在 shell 裡則用 `hapi spawn-peer`，再輪詢 `hapi inspect-peer <id> --json`。
- 收完結果就 archive 自己開的 reviewer：Hub API `POST /api/sessions/<id>/archive`。0.35 沒有 `hapi archive-peer`（回 Unknown command）；archive 後用 `inspect-peer` 確認 `active=false` 才算收掉，不要沒驗證就回報。
- zsh 的 `set -- $s` 不分詞；從 JSON 取 ID 用 `bash -c` 搭配 `read ... < <(python3 ...)`。
- 0.30.x 的舊經驗：`spawn-peer` 偶爾卡十幾分鐘，用 `timeout 180` 包起來並帶新的 `--remit-id` 重試；`wait-peer` 會 ECONNRESET，要在迴圈裡重試。0.35 已沒有 `--remit-id` 和 `wait-peer`。

## 各模型的審查品質（這一次）

- Codex（gpt-6.1-sol，high）：6 條全是真問題，含 2 條 PR 引入的記憶體預算錯誤，附具體數字的失敗情境。
- Claude：2 條真問題（`rs -K 1` 越界、`rsz` 的 `-z` 沒作用），並列出檢查過、確認乾淨的地方，對判斷很有用。
- Gemini（agy）與 GitHub Gemini bot：critical/high 多為誤報（宣稱編譯錯誤、lambda 抓錯變數、越界），需逐條對照原始碼。
- 所有發現都要自己對照最新 head 驗證後再交給修正的人。
