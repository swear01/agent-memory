---
title: Swear Review 因 Go 訂閱失效 403 中斷備援的修復
scope: tools/review deepseek-latch-gateway and Swear Review on Oracle
status: fix merged; Oracle deployed and end-to-end verified; other hosts not deployed
updated: 2026-10-12
---

## 原因與責任邊界

2026-10-12 台灣時間查驗時，OpenCode Go 帳號 1 回 HTTP 429，
`GoUsageLimitError` 的 `limitName` 為 `monthly`；帳號 2 回 HTTP 403：
`Upstream request failed: An active OpenCode Go subscription is required to use Go models.`
帳號 3 能完整回答 `OK`，Command Code 與 OpenRouter 的小型請求也回 200。
這些是當時的帳號狀態，之後診斷必須重新測量。

Gateway 原本沒有辨識此訂閱錯誤，會直接轉送帳號 2 的 403，沒有繼續試帳號 3。
修復放在 Gateway 共用的 `isRateLimitOrQuotaError`：僅在 HTTP 403 且包含上述
訂閱要求訊息時，沿用 quota failover 與 cooldown；priority 與 flat RS-latch
都受益。一般 permission-denied 403 仍原樣回傳。Swear Review 不必另加重試或
自行切帳號；重啟服務也不能補回月額度或恢復訂閱。

## 已驗證修復

- `swear01/deepseek-latch-gateway` PR #18 已合併；merge commit
  `9d5f2bf294af7cf83a1e785962a840b03590cba6`，review head
  `01bafeb4c8dab570328af1c25dd1dd9dca81b143`。
- 新測試先重現「429 → 訂閱失效 403 → 未嘗試健康帳號」；修正後 `bun test`
  65 tests、typecheck、四平台建置與 Mac 簽章驗證、GitHub CI 全部通過。
  測試同時確認一般 403 不切換、穩定 session header 與 priority cooldown。
- Oracle 隔離候選 port 35011 與正式 port 35001 均完成
  「帳號 1 quota → 帳號 2 inactive subscription → 帳號 3」；HTTP 200、
  `X-Gateway-Active-Endpoint: opencode-go-3`、`X-Gateway-Attempt: 3`，回答 `OK`。
- Linux ARM64 正式 binary 與實際執行程序 SHA-256 一致：
  `2d82b6a680444f46847cdbe6c4447e7d25b812b1a8a8ccfa56e39321cc288691`。
  config、routing、launcher、systemd unit hash 不變，保留已驗證舊 binary 備份；
  Swear Review 與 HAPI Runner 沒有重啟。
- 同一 PR head 的 Swear Review 在 26 秒完成：selected 1、completed 1、failed 0、
  findings 0，並成功發布 GitHub summary。這是完整鏈路恢復的證據；
  舊 Valheim CLI PR #12／#13 的失敗紀錄沒有因此被重跑或改寫。

## 操作與驗證界線

- 本次只部署 Oracle；其餘七台沒有更新，不能宣稱 fleet rollout 完成。
- `/healthz`／`/readyz` 只證明程序與資料庫存活，必須另核對實際模型回應、
  upstream header、OCR coverage 與 GitHub 發布。Gate off 的綠燈也不等於零 findings。
- 必須等 `scp` 完成並核對完整檔案 SHA-256，才可啟動候選程式。邊上傳邊執行
  曾造成候選 hash 不符與 `Text file busy`；這不是候選邏輯失敗。
- 只在相關 review queue 與 Gateway established 連線清空後原子替換，重啟 Gateway
  即可；保留設定、憑證、其他服務和使用者工作階段。
