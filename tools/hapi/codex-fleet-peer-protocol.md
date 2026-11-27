---
title: HAPI 0.35 Codex 機群：子 session 回覆自動回送、mazu 啟動逾時、改模型 API
scope: tools/hapi
status: active
updated: 2026-11-27
tags: [hapi, codex, spawn-peer, ping-peer, waitForReply, model-switch, mazu, nfs]
---

# Codex 子 session 的回覆會自動回到呼叫者（2026-11-27 valkyrie 實測）

- Codex session 在 HAPI 下看得到 MCP 工具 `spawn_peer`、`ping_peer`、`inspect_peer`。用 `spawn_peer(..., waitForReply=true)` 派工後，子 session 一結束當輪，HAPI 就把最後一則訊息排進**呼叫者**的對話。追問時用 `ping_peer(waitForReply=true, deliveryMode=queue)`，回覆一樣會自動送回。
- 實測 1,388 個 ASCII 字元全文送達，SHA 相同，結尾標記完整。`inspect_peer` 的預覽只截到 1,200 個 UTF-16 單位，所以不能拿預覽當全文。
- Parent 標記不會訂閱後續回合，CLI 的 `--wait` 也只等到啟動或投遞完成。只用 CLI 時，要約定子 session 完成後主動 `ping-peer <parent> --delivery-mode queue` 一次。
- 呼叫端 CLI 每 10 秒檢查一次訂閱，最長 6 小時。CLI 一重啟，等待中的回覆就會遺失，這時要依任務表逐一 `inspect-peer` 未結案的 session 補收。
- 實用協議：
  - 每個任務給一個 ID；
  - 子 session 只回一行 `ID | DONE/WAITING/BLOCKED | commit | 結果 | report 路徑`；
  - 長報告推進 git；
  - 主控自己維護一份 ledger，子 session 的自我回報要自己核對過才算數。

# mazu 上 Codex 起不來（`engine.sock ENOENT`／`startup is still unconfirmed`）

HAPI 0.35.0.1 的 engine 初始化與 app-server RPC（`thread/start`）各有 20 秒期限。mazu 的 NFS `/home` 當時每秒有約 1,131 次 `TEST_STATEID`，`GETATTR` 平均排隊 18 ms，啟動因此超過期限；同一個 worktree 在 valkyrie 上 1.4 秒就收到 webhook。換句話說，**kernel 7.0.0-43 之後 mazu 又出現了 TEST_STATEID 風暴**，見 [[machines/cthulhu/nfs-test-stateid-storm]]。在 NFS 恢復正常前，Codex peer 先開在 valkyrie。失敗的 session 無法 resume，只能封存。

# 改 Codex session 的模型

- `POST /api/sessions/<id>/model {"model":"gpt-6.2-sol"}` 可以用。同時帶 `effort` 會回 400：`Effort can only be sent with a model change for Claude and Antigravity sessions`。
- effort 要另外打 `POST /api/sessions/<id>/model-reasoning-effort {"modelReasoningEffort":"high"}`。
- session 正忙時，兩個 API 都可能回 409 `Native settings update is not confirmed; reconnect before retrying`。等它閒下來再改，或直接用 `spawn-peer --model ... --effort ...` 重開。
- 要確認模型有沒有生效，讀 `GET /api/sessions/<id>`，看 `model` 和 `modelReasoningEffort`。程序參數只反映啟動時的值；沒帶 `--model` 啟動的 session 用的是該機 `~/.codex/config.toml` 的預設。
- JWT 的取得方式：`POST /api/auth {"accessToken": <cli token>}`。可以借用 `hapi-session-control/scripts/archive-peer.py` 的 `call()`/`token()`。
