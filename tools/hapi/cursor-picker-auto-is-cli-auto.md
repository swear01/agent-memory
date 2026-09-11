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
updated: 2026-09-11
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
- 若 process 是用 `--model auto` spawn 的，可承認 live Auto；一旦成功切到具體模型，就要清掉這個 spawn 假設，否則 Auto → concrete → Auto 會假切換。

# 驗證

看 spawn argv 是否含 `--model auto`，以及 hub session.model 是否為 `auto`。不要把 ACP `default[]` 或省略 `--model` 當成 Auto 成功。
