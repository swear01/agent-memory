---
title: DeepSeek Latch Gateway fleet routing and automatic failback
scope: projects/deepseek-latch-gateway
project: deepseek-latch-gateway
tool: Bun/systemd/launchd/Windows-Task-Scheduler
status: active
created: 2026-08-25
updated: 2026-09-07
tags: [gateway, opencode-go, routing, failback, hapi, swear-review]
---

# 架構

Gateway 是分散式部署，不是中央單點：每個 client 都呼叫同機 loopback 上的
gateway process。現行 fleet 共八台，Mac、mazu、athena、cthulhu、valkyrie、Oracle、swop 使用
`35001`；Zeus 因為同機另一帳號會占用 `35001`，所以使用 `35002`。

`routing.yaml` 只定義 model 到 priority group 的資料；gateway binary 內的
`PriorityLatchManager` 才負責 fallback、cooldown、half-open probe 與自動
failback。因此 routing code 修正必須部署 binary，只改 route file 不足以生效。

# 部署不變量

- Mac 使用獨立 arm64 binary 與 LaunchAgent。
- mazu、athena、cthulhu、valkyrie 透過 NFS home 共用同一份 x64 binary，
  但每台有自己的 process，仍須逐台 restart。
- Oracle 使用獨立 arm64 binary 與 systemd user service。
- Zeus 使用獨立 x64 binary 與 systemd user service。
- swop 使用 Windows x64 binary，正式位置為 `<program-data>/DeepSeekGateway/deepseek-gateway.exe`，由 SYSTEM Scheduled Task `DeepSeek Gateway (SWOP)` 管理；使用者目錄 `.local/bin/deepseek-gateway.exe` 副本也須同步。
- Gateway 清單不能從舊 HAPI／Linux 清單推測；必須包含 swop 並核對真實 supervisor。
- 更新 Swear Review 的 caller header 不代表 Gateway 已更新；兩者是不同部署。
- 先在 live path 之外完成 staging 並核對 SHA，再以 atomic replacement
  換入正式路徑、保留已知可用備份，最後 restart。
- 文件與記憶不得保存 API key 值。

# Recovery 行為

Quota failure 會先依序耗盡三個 OpenCode Go account，再進入低優先級 route。
Cooldown 從 1.5 小時開始，failed recovery probe 後倍增，最高 24 小時。
到期後由一個真實 request 擁有 half-open probe；其他 concurrent request
繼續使用 fallback。Probe 成功便恢復到可用的最高 priority。

# 歷史 rollout（七台，非現行清單）

2026-08-25 已對 commit `0944c14` 完成 35 tests、typecheck、三平台 build，
並部署至七個 gateway。Live check 驗證 service health、各平台 binary SHA、
Priority 1 route 載入與第三個 OpenCode endpoint 存在。Oracle 原本已是相同
arm64 build，因此沒有做多餘 restart。正式環境沒有刻意耗盡 quota 等待
1.5 小時；自動 failback 時序由 unit 與 integration tests 驗證。

再次使用這份紀錄前，仍須重查 live hash、service、config 與 endpoint status。

# 2026-09-07 八台 session header rollout

PR #10 合併提交 `73f0fb5`（review head `ae9963c`）保留既有 priority routing，
修正共用 forwarder 的 OpenCode POST header。46 tests、typecheck、四平台
建置通過；Swear Review 對該 head 為 0 findings。八台 binary、程序及 health
均已驗證，包括最初漏掉、後續補部署的 swop；config/routing 保持不變。

- 保留 caller 的 `x-opencode-session`，優先於 endpoint 靜態 header。
- 缺少時依序接受 `x-deepseek-harness-session-id`、`x-session-id`、`x-session-affinity`、`session_id`。
- 完全沒有 ID 時，以 client identity 與開場 user message 派生 SHA-256 affinity；重試及保留開場的後續對話保持相同值。
- 相同 client 與相同開場會共用 affinity；compaction 更換開場會改變它。精確隔離需要 caller 傳入穩定 ID，不能把雜湊 fallback 說成可靠 conversation identity。
- 同時沒有 ID 與開場時，OpenCode 路徑回傳本地 HTTP 400 `missing_session_id`；其他 provider 不變。

隔離的原生封包測試驗證 header、alias、重試及多輪穩定性。八台真實推論
皆 HTTP 200，但實際走 CommandCode fallback；這證明 gateway inference
可用，不證明 OpenCode upstream inference 成功。不可混用兩種驗證證據。

Linux 應比對正式 binary 與 `/proc/<pid>/exe` 的 SHA-256；Mac、Windows
則核對新程序的 executable path、啟動時間與已安裝 binary hash。保留舊版
備份，等 established 連線清空後只重啟 Gateway，不重啟 HAPI Runner。
