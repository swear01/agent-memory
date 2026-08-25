---
title: DeepSeek Latch Gateway fleet routing and automatic failback
scope: projects/deepseek-latch-gateway
project: deepseek-latch-gateway
tool: Bun/systemd/launchd
status: active
created: 2026-08-25
updated: 2026-08-25
tags: [gateway, opencode-go, routing, failback, hapi, swear-review]
---

# 架構

Gateway 是分散式部署，不是中央單點：每個 client 都呼叫同機 loopback 上的
gateway process。Mac、mazu、athena、cthulhu、valkyrie、Oracle 使用
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
- 先在 live path 之外完成 staging 並核對 SHA，再以 atomic replacement
  換入正式路徑、保留已知可用備份，最後 restart。
- 文件與記憶不得保存 API key 值。

# Recovery 行為

Quota failure 會先依序耗盡三個 OpenCode Go account，再進入低優先級 route。
Cooldown 從 1.5 小時開始，failed recovery probe 後倍增，最高 24 小時。
到期後由一個真實 request 擁有 half-open probe；其他 concurrent request
繼續使用 fallback。Probe 成功便恢復到可用的最高 priority。

# 已驗證 rollout

2026-08-25 已對 commit `0944c14` 完成 35 tests、typecheck、三平台 build，
並部署至七個 gateway。Live check 驗證 service health、各平台 binary SHA、
Priority 1 route 載入與第三個 OpenCode endpoint 存在。Oracle 原本已是相同
arm64 build，因此沒有做多餘 restart。正式環境沒有刻意耗盡 quota 等待
1.5 小時；自動 failback 時序由 unit 與 integration tests 驗證。

再次使用這份紀錄前，仍須重查 live hash、service、config 與 endpoint status。
