---
title: HAPI Cursor picker Auto 是 CLI auto，不是帳號 default 也不是 ACP default[]
scope: tools/hapi
tool: HAPI Cursor ACP launcher
status: verified
confidence: high
evidence: >-
  tiann/hapi#1817 / #1819. `agent --list-models` 有 `auto - Auto`。
  舊路徑把 picker Auto、省略 --model、ACP default[] 收成同一個 automatic token。
  #1247/#1248 只把 Default 標籤改名成 Auto，行為沒改。
created: 2026-09-11
updated: 2026-09-14
tags:
  - hapi
  - cursor
  - auto
  - model-picker
  - acp
source_refs:
  - hapi:shared/src/cursorCliSku.ts
  - hapi:cli/src/cursor/utils/cursorAcpBackend.ts
  - hapi:cli/src/cursor/cursorAcpRemoteLauncher.ts
  - hapi:web/src/lib/sessionChatCursorModel.ts
related:
  - tools/hapi/cursor-acp-parameterized-model-wire-drops-effort.md
redaction: passed
---

# 三個不同的東西

HAPI Cursor picker 的 **Auto** 必須對到 CLI `agent --model auto`（`agent --list-models` 的 `auto - Auto`）。下面三個不是同一件事：

1. CLI `auto`：spawn 帶 `--model auto`，session.model persist `'auto'`。
2. 省略 `--model`：帳號 / `~/.cursor/cli-config.json` default。
3. ACP `default[]`：live configOptions 裡名叫 Auto 的帳號 default wire。

# 舊雷

- #1247 / #1248 只把 picker 標籤 Default → Auto。新 session 選 Auto 送 `undefined`；進行中選 Auto 送 `null`，再 `set_config_option` 成 `default[]` 並清掉 stored model。
- 不要再用「選 Auto 來避開壞掉的 parameterized wire、改走帳號 default」當 workaround；那會讓使用者以為在跑 CLI Auto。

# 修正後行為（#1819）

- 新 Cursor session 選 Auto：persist 並 spawn `--model auto`。其他 flavor 的 `auto` 仍可 omit。
- Picker 沒有 Default/unset 列。`auto` / `default` / `default[]` 都正規成 CLI `auto`。
- 進行中 session：ACP catalog 若有字面 `auto` 才 live `set_config_option`。若只有 `default[]`，拒絕 live Auto（需要重啟才能套 `--model auto`），不要假裝已切換、也不要把 Auto 收成 `default[]`。
- 若 process 是用 `--model auto` spawn 的，可承認仍在使用 Auto；一旦具體模型完全或部分套用，就要清掉這個 spawn 假設。跨客戶端的模型請求也必須依序執行，否則 Auto → concrete → Auto 會假切換。

# 驗證

看 spawn argv 是否含 `--model auto`，以及 hub session.model 是否為 `auto`。不要把 ACP `default[]` 或省略 `--model` 當成 Auto 成功。

## 2026-09-14 範圍決策與失敗路徑

使用者明確選擇保留新建／resume 修正，選單明示即時切換限制；不自動重啟使用中的 session，也不在這個 PR 新增重啟流程。Session picker 在原生 session catalog 缺少字面 `auto` 時提示「切回需要重啟」；machine catalog 的 Auto 不能冒充 live capability。新建選單不套用這個限制提示。

- 參數式套用可能先成功切 base，再於 fast／effort 失敗。Helper 回報 `partiallyAppliedWireId`，launcher 清除 Auto 假設、同步已確認的模型狀態，再回報失敗；不能回滾顯示成仍在 Auto。
- 沿用 `AsyncLock` 逐一處理所有 live model request，包括 Auto 與 session sync。只靠前端單一 client 的 queue 或 last-writer sequence 不足；另一個 client 可能在具體模型尚未完成時得到假的 Auto 成功回覆。
- 隔離真實 CLI probe 驗證 Auto spawn、最小回合、具體模型與 effort 切換、同一 session resume。空白且未執行過 turn 的 session 曾被 loadSession 拒絕；完成最小回合後 resume 成功。ACP 回報 `default` 本身不是 CLI Auto 證據。
- 明確使用 `cursor-agent` 並核對 executable；此機器裸 `agent` 曾解析成 Grok，不能因命令名稱相同就當成 Cursor。

證據：#1819 `eb22b2af` 整合 #1822 `5b78b731`；當時最新 head CI 與 review 均通過，仍為未合併 upstream PR。之後使用必須重查版本與 head，不視為 production 已部署。
