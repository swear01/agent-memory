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

mazu 再出現 `engine.sock ENOENT` 或 `startup is still unconfirmed` 時，先依回報的 PID 確認原 session 是否仍在啟動，再檢查正式 Runner 的 launcher 與 `hapi-mazu-codex.socket`。修復後用 `spawn-peer --machine mazu --agent codex --permission-mode autoReview` 新建測試 session，等第一則模型回覆。HAPI v0.36.7.1 的 `thread/start` RPC 等 20 秒，NFSv4.1 大量發出 `TEST_STATEID` 時，技能和外掛目錄掃描可超過此期限。底層診斷與非 NFS 主機對照見 [NFS TEST_STATEID 風暴](../../machines/cthulhu/nfs-test-stateid-storm.md)。

- 主機的 `/home` 仍是 NFSv4.1。HAPI Codex worker 的私有 mount namespace 只把 `/home` 換成同一個 NAS 的 NFSv4.0 掛載，沿用正式 Runner 原本的 `CODEX_HOME`、群組和本機 SQLite。技能、內建 `skills/.system`、外掛與 rollout 都在原路徑，技能目錄本身可以被 Skillshare 更換。
- 驗封存相容性要選本次工作自己在部署前用原 profile 建立的測試 session。試行版的 profile 別名雖指向相同 inode，Codex 仍因 canonical rollout 路徑不在它認定的封存目錄下而回報 `must be in archived directory`。改回原 `CODEX_HOME` 後，同一個舊 session 以原 native thread ID 恢復，模型能從前文答出 `MAZU-CODEX-OK`。只驗部署後新建的 session 會漏掉這項回歸。
- 檢查真正的冷啟動時，在排空 worker 後卸載、重新掛載 NFSv4.0。首次未預熱時，約 20 秒仍找不到 `engine.sock`。冷掛載量到 `codex --version` 花 8.65 秒，緊接著啟動隔離 app-server 又花 12.43 秒。部署的 `hapi-mazu-codex-warm.service` 在每次 backing mount 啟用後，先用私有暫存 profile 讀入 Codex 程式，stdin 給 EOF。socket 等到 oneshot 成功才開始監聽。重新掛載後，正式 Runner 收到新 session webhook 用了 25.086 秒，接著收到第一則模型回覆。mount 與 socket 已啟用開機啟動，尚未實測整機重開。
- 回滾先撤除 Runner 的 launcher drop-in，確認新啟動走原 CLI，等既有 worker 與連線排空，再停 socket、warm service 與 NFSv4.0 掛載。曾部署別名試行版的 mazu 保留原有 `/var/lib/hapi-codex/profile` symlink，讓 SQLite 中記錄的試行版 rollout 路徑仍可讀。全新安裝不建立該別名。這次驗證時 PR #109 仍在審查。[部署、回滾與驗證](https://github.com/swear01/transfer_MAC/pull/109)

# 改 Codex session 的模型

- `POST /api/sessions/<id>/model {"model":"gpt-6.2-sol"}` 可以用。同時帶 `effort` 會回 400：`Effort can only be sent with a model change for Claude and Antigravity sessions`。
- effort 要另外打 `POST /api/sessions/<id>/model-reasoning-effort {"modelReasoningEffort":"high"}`。
- session 正忙時，兩個 API 都可能回 409 `Native settings update is not confirmed; reconnect before retrying`。等它閒下來再改，或直接用 `spawn-peer --model ... --effort ...` 重開。
- 要確認模型有沒有生效，讀 `GET /api/sessions/<id>`，看 `model` 和 `modelReasoningEffort`。程序參數只反映啟動時的值；沒帶 `--model` 啟動的 session 用的是該機 `~/.codex/config.toml` 的預設。
- JWT 的取得方式：`POST /api/auth {"accessToken": <cli token>}`。可以借用 `hapi-session-control/scripts/archive-peer.py` 的 `call()`/`token()`。
