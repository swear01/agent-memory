---
title: HAPI Codex 機群：子 session 回覆、mazu NFS 啟動入口、改模型 API
scope: tools/hapi
status: active
updated: 2026-11-28
tags: [hapi, codex, spawn-peer, ping-peer, waitForReply, model-switch, mazu, nfs]
---

# Codex 子 session 的回覆會自動回到呼叫者（2026-11-27 valkyrie 實測）

- Codex session 在 HAPI 下看得到 MCP 工具 `spawn_peer`、`ping_peer`、`inspect_peer`。用 `spawn_peer(..., waitForReply=true)` 派工後，子 session 一結束當輪，HAPI 就把最後一則訊息排進**呼叫者**的對話。追問時用 `ping_peer(waitForReply=true, deliveryMode=queue)`，回覆一樣會自動送回。
- 實測 1,388 個 ASCII 字元全文送達，SHA 相同，結尾標記完整。`inspect_peer` 的預覽只截到 1,200 個 UTF-16 單位，所以不能拿預覽當全文。
- 上游 `6f7420aa6` 的 `ping_peer(auto)` 在送出前用最新 session 狀態選一次 Queue 或 Steer。忙碌中的 Claude/Codex 若未回報 `steeringActive`，這筆會排隊，tool 之後不會再提升它。`waitForReply` 回傳到父 session 時則固定走 Queue，與去程的 deliveryMode 無關。查個案時先看方向、`localId` 與 `steerNote`。[去程](https://github.com/tiann/hapi/blob/6f7420aa6/cli/src/modules/pingPeer/pingPeer.ts#L800-L827) [回程](https://github.com/tiann/hapi/blob/6f7420aa6/cli/src/modules/pingPeer/pingPeer.ts#L1293-L1302)
- Parent 標記不會訂閱後續回合，CLI 的 `--wait` 也只等到啟動或投遞完成。只用 CLI 時，要約定子 session 完成後主動 `ping-peer <parent> --delivery-mode queue` 一次。
- 呼叫端 CLI 每 10 秒檢查一次訂閱，最長 6 小時。CLI 一重啟，等待中的回覆就會遺失，這時要依任務表逐一 `inspect-peer` 未結案的 session 補收。
- 實用協議：
  - 每個任務給一個 ID；
  - 子 session 只回一行 `ID | DONE/WAITING/BLOCKED | commit | 結果 | report 路徑`；
  - 長報告推進 git；
  - 主控自己維護一份 ledger，子 session 的自我回報要自己核對過才算數。

# mazu 的正式 HAPI Codex 啟動入口

mazu 再出現 `engine.sock ENOENT` 或 `startup is still unconfirmed` 時，先依回報的 PID 確認原 session 是否仍在啟動，再檢查正式 Runner 的 launcher 與 `hapi-mazu-codex.socket`。修復後用 `spawn-peer --machine mazu --agent codex --permission-mode autoReview` 新建測試 session，等第一則模型回覆。HAPI v0.36.7.1 的 `thread/start` RPC 等 20 秒，NFSv4.1 大量發出 `TEST_STATEID` 時，技能和外掛目錄掃描可超過此期限。底層診斷見 [[machines/cthulhu/nfs-test-stateid-storm]]。

- 主機的 `/home` 繼續使用 NFSv4.1。只有 HAPI 新開的 Codex app-server 在私有 mount namespace 以 NFSv4.0 存取同一個 NAS home，保留原使用者與群組。共享的設定、外掛及 session 歷史仍在原處。使用預設 profile 時，worker 在別名 `/var/lib/hapi-codex/profile` 下用 tmpfs 放目前 Codex 版本的內建 `skills/.system`。原本的 Skillshare 與 Codex 目錄沒有額外的掛載點，因此 Skillshare 仍可更換整個目標目錄。
- 2026-11-28 在原有的正式 user Runner 和 Codex 0.172.0 驗證。重啟 Runner 前後，新 session 的 webhook 分別在 7.677、15.998 秒收到，兩次都收到了第一則模型回覆。既有執行中 session 跨過 Runner 重啟，封存的測試 session 也能 resume 並回覆 `MAZU-RESUME-OK`。system mount 與 socket 已啟用開機啟動，尚未實際重開 mazu 驗證。
- 回滾時先撤除 Runner 的 launcher drop-in，確認新啟動已走原 CLI，等既有 worker 和連線排空，再停止 socket 與 NFSv4.0 掛載。保留主機上的 `/var/lib/hapi-codex/profile` symlink，本機 SQLite 記錄的 rollout 絕對路徑仍可能經此別名讀回。部署與回滾指令見 transfer_MAC PR #109 的 README，這次驗證時程式仍在審查中。[操作文件與驗證](https://github.com/swear01/transfer_MAC/pull/109)

# 改 Codex session 的模型

- `POST /api/sessions/<id>/model {"model":"gpt-6.2-sol"}` 可以用。同時帶 `effort` 會回 400：`Effort can only be sent with a model change for Claude and Antigravity sessions`。
- effort 要另外打 `POST /api/sessions/<id>/model-reasoning-effort {"modelReasoningEffort":"high"}`。
- session 正忙時，兩個 API 都可能回 409 `Native settings update is not confirmed; reconnect before retrying`。等它閒下來再改，或直接用 `spawn-peer --model ... --effort ...` 重開。
- 要確認模型有沒有生效，讀 `GET /api/sessions/<id>`，看 `model` 和 `modelReasoningEffort`。程序參數只反映啟動時的值；沒帶 `--model` 啟動的 session 用的是該機 `~/.codex/config.toml` 的預設。
- JWT 的取得方式：`POST /api/auth {"accessToken": <cli token>}`。可以借用 `hapi-session-control/scripts/archive-peer.py` 的 `call()`/`token()`。
