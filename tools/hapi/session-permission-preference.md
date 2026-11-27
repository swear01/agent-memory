---
title: HAPI 新 session 漏傳 Auto 會卡在工具核准
scope: tools/hapi
status: active
updated: 2026-11-28
created: 2026-08-26
---

新建 HAPI coding session 時明確傳 permission mode。Claude 包含審查用 `auto`，Codex 用 `autoReview`，其他 agent 核對各自支援的值。MCP 用 `permissionMode`，CLI 用 `--permission-mode`。只有使用者當次指定時才改用其他模式。這個參數只改核准方式，任務範圍由原授權決定。

- **`thinking=true` 卻沒有輸出或檔案變更。** 先查這輪的 permission request。Claude SDK 的 `default` 即 Manual，互動式 Claude 的預設與 `settings.json` 無法覆蓋明確的 SDK 旗標。部分 spawn 路徑或版本的省略行為也曾落到 YOLO，不能靠省略推測實際模式。
- **已切成 Auto，第一個工具仍卡住。** 維護版 0.30.7.7 已驗證 `hapi set-mode <exact-session-id> auto`，只影響之後的呼叫。若舊請求仍 pending，先依該版本中斷確切 session 的本輪，再 `ping-peer` 續跑，核對 requests 清空。HAPI 0.35 的 CLI 已移除 `abort-peer`，用 Web 中斷。封存或重開前先核對 session 和既有工作。
- **收到 `Create Unsafe Agents`。** Claude 的 `bypassPermissions` 是 YOLO，`dontAsk` 是自動拒絕需要詢問的呼叫。依使用者既有偏好傳 Auto，再讀回 agent、model、machine、workspace、permission mode 與 ready。
- 0.35 的工具和封存路徑見 [peer CLI 對照](peer-cli-0-35-command-changes.md)。驗證 Skillshare 部署時，查發布 commit 是 source 的 ancestor，再查實際 target 是否包含規則；乾淨的 detached merge 可合法存在。

[原始診斷與查證紀錄](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/tools/hapi/session-permission-preference.md)
