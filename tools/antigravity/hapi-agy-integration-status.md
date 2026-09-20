---
title: HAPI × Antigravity (agy) 整合狀態與登入前提
scope: tools/antigravity
tool: Antigravity CLI (agy)
status: active
confidence: high
evidence: >-
  2026-09-20 在 mazu/zeus 對上游 tiann/hapi 與 fork swear01/hapi 交叉查證：
  上游 PR/issue、release audit TSV、hub DB sessions metadata、runner log，
  以及 `agy --output-format=json models` 與 `agy -p` 的實際輸出。
created: 2026-09-20
updated: 2026-09-20
tags:
  - antigravity
  - agy
  - hapi
  - integration
---

## 整合架構（HAPI 端）

- agy 是 HAPI 官方支援的 agent（`hapi agy`、web 選 Antigravity）。上游
  `tiann/hapi` PR #1320（2026-08-04 合併）先以 PTY 包 TUI，PR #1591
  （2026-08-16 合併，本機 owner 提出）改成 **headless print-mode transport**，
  刪掉約 8k LOC 的 PTY/鍵盤/畫面 scraping，設計文件在
  `docs/design/agy-headless-transport.md`（以 release tag 為準）。
- 每回合固定 spawn 一次：
  `agy -p <msg> --output-format stream-json --conversation <uuid>
  --model <slug> --mode accept-edits|plan --effort low|medium|high
  --print-timeout 30m [--dangerously-skip-permissions]`，解析 NDJSON 的
  init / step_update / result；conversation id 取自 init envelope，後續回合用
  `--conversation` resume。
- **agy 只能 remote mode**：hub 的 spawn 路由對 `startingMode !== 'remote'`
  回 400，因此沒有 terminal 檢視、沒有本機 PTY 模式。
- 模型清單來自 `agy --output-format=json models`（含文字表格與硬編鏡像的
  fallback）；in-session 切換模型會讀同一份 machine catalog（#1814）。
- Session 標題從 `~/.gemini/antigravity-cli/conversation_summaries.db` 同步
  （#1476）；spawn 時會濾掉 `SSH_CLIENT`/`SSH_CONNECTION`/`SSH_TTY`。

## Fork（swear01/hapi）政策

- #1320 在 audit ledger 一直是 `drop`，但那只表示「不重放 carry patch」；
  官方 base 自 v0.26.0.1 起已含它，fork build 因此一直有 agy 支援。
- v0.30.7.1 之前的 audit 都把兩個 agy 修復 PR 列為 `defer`：upstream #1629
  （delta UTF-8 破損造成長非 ASCII 答案重複送兩次，仍有 HAPI Bot findings）、
  #1642（失敗回合把答案當錯誤理由，應讀 `error` 欄位）。目前 fork 不含這兩個
  修復，且上游也未合併。

## 機隊操作前提：agy 必須在共用 NFS home 登入

- Linux runner 的 `$HOME` 是 NFS 共享（`192.168.1.200:/volume1/nfs-home`）。
  `~/.gemini/antigravity-cli` 與 token 檔
  `~/.gemini/antigravity-cli/antigravity-oauth-token` 都在這個 home 內，
  所以**登入一次即可覆蓋所有掛同一 home 的 Linux runner**。
- 2026-09-20 實測 mazu 與 zeus 都未登入：`agy --output-format=json models`
  回 `Please sign in to view available models`，`agy -p "..."` 直接印出
  Google OAuth URL 並等待 60 秒，token 檔不存在。
- 未登入時 HAPI 端不會報錯給使用者：runner log 只看得到
  `[agy-headless] Starting headless driver`、`List Agy models request` 與
  cleanup，hub DB 會留下 0 筆 message 的 agy session（mazu 2026-09-20 三筆
  即此情況）。反之，Mac（459 messages，2026-09-19~20）與 oracle
  （192 messages，2026-09-11）有真實對話，代表那兩台已登入。
- 驗證真實登入狀態不要只看 log 的 `You are not logged into Antigravity`
  （啟動競態會誤報），要用功能驗證：`agy -p "reply only with OK"` 或
  `agy --output-format=json models`。

## 快速查詢

```bash
# 每個 flavor 的 session 數（hub DB，metadata 是 JSON）
python3 -c "import sqlite3,json,collections;c=sqlite3.connect('file:<hub-home>/hapi.db?mode=ro',uri=True);print(collections.Counter(json.loads(m or '{}').get('flavor') for _,m in c.execute('select id,metadata from sessions')))"

# runner log 中的 agy 活動
grep -E '\[agy|Agy' "<runner-home>/logs/"*runner.log
```

真實 agy session 的特徵：`metadata.flavor == 'agy'`、messages 表有實際
rows、runner log 有 `[agy-headless]` 的 main loop 與 cleanup；0 rows 要懷疑
登入或模型探測失敗。
