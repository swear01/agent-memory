---
title: 用 HAPI 派多個模型做程式審查
scope: tools/hapi
tool: hapi spawn-peer
status: active
updated: 2026-10-30
tags: [hapi, review, codex, claude, agy, gemini, cursor, grok, dsh, permission-mode]
---

# 用 HAPI 派多個模型做程式審查

2026-10-29 在 CULS-fast PR 上用 HAPI 派 Codex、Claude、Gemini（agy）、OpenCode、Cursor、Grok、DeepSeek（dsh）做靜態審查。

## 權限模式：無人看管的審查要開高權限

使用者明確要求：「session 要給他比較高的權限等級，他才能自己好好運作」，多個 session 開在同一資料夾也可以。這是針對無人看管的審查；一般 session 仍照 [[session-permission-preference]] 用 Auto。

`--permission-mode` 合法值（取自 `hapi` binary 字串）：

| agent | 高權限值 | 備註 |
|---|---|---|
| claude | `bypassPermissions` | 省略時是 `default`（Manual），每步等核准 |
| codex | `yolo` | `read-only` 連 FINDINGS.md 都寫不了 |
| agy（Gemini） | `always-proceed` | 另一個是 `request-review` |
| cursor | `yolo` | |
| grok | `bypassPermissions` | 不接受 `yolo`（bad_args） |
| opencode | `yolo` | |
| dsh（DeepSeek） | 無此旗標 | |

教訓：使用者提高權限要求後，只改了新開的 session，先前用 `read-only`（Codex）和 `default`（Claude）開的兩個繼續跑了幾小時、零產出，使用者很不滿。要求一改，就檢查所有執行中的 session（`spawn_*.json` 的 `session.permissionMode` 或 `hapi inspect-peer`），模式不對就 archive 後用同一份 remit 重開。開完立刻確認 `permissionMode`。

## 登入狀態（2026-10-29）

- agy 只在 Mac（swairM5）登入；mazu、cthulhu、zeus 沒有。
- cursor-agent 各機都沒登入，spawn 回 HTTP 502；要使用者 `agent login`。
- grok 只裝在 Mac 且未登入（`grok login`）。
- dsh 在 mazu 沒有 `DEEPSEEK_API_KEY`（`dsh --profile headless` 回 MISSING_CREDENTIAL）。使用者 2026-10-30 決定不用 dsh；DeepSeek 改用 OpenCode：`--agent opencode --model deepseek/deepseek-v4-pro --permission-mode yolo`（Mac 與 mazu 的 opencode 都已綁好 DeepSeek）。
- CLI 沒登入時 HAPI session 會無聲結束（`session_ended`）；先用 headless 一行指令測試。

## 操作細節

- `hapi spawn-peer` 偶爾卡住十幾分鐘不回；用 `timeout 180` 包起來並帶新的 `--remit-id`，失敗就重試。
- `hapi wait-peer` 會 ECONNRESET 或 socket hang up，要在迴圈裡重試。
- zsh 的 `set -- $s` 不分詞；讀 sessionId/remitId 用 `bash -c` 搭配 `read sid rid < <(python3 ...)`。
- 收完結果就 `hapi archive-peer`。

## 各模型的審查品質（這一次）

- Codex（gpt-6.1-sol，high）：6 條全是真問題，含 2 條 PR 引入的記憶體預算錯誤，附具體數字的失敗情境。
- Claude：2 條真問題（`rs -K 1` 越界、`rsz` 的 `-z` 沒作用），並列出檢查過、確認乾淨的地方，對判斷很有用。
- Gemini（agy）與 GitHub Gemini bot：critical/high 多為誤報（宣稱編譯錯誤、lambda 抓錯變數、越界），需逐條對照原始碼。
- 所有發現都要自己對照最新 head 驗證後再交給修正的人。
