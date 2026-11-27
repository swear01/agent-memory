---
title: Session 清冊漏算與錯把在線當成工作中
scope: global/workflows
status: active
updated: 2026-11-28
created: 2026-08-20
---

- **清單全部封存，卻還有未列入的聊天。** Dispatch ledger 只證明已登記的 sessions。用成功 spawn receipt 記 parent、child、remit，失敗後重開也各自登記。使用者給出確切 session ID 時單獨核對。登記 worktree 的 bounded rollout 掃描可補找孫 session，回報涵蓋日期、來源及漏看的 form。
- **Runner 還在線但 `thinking=false`。** 分開查 lifecycle、當輪結果與 owned background process。保存 handoff、驗收實際產物再封存；移除 worktree 前核對每個 parent／child 的 CWD。若 CWD 早已消失，用終止證據與程序檢查完成收尾，不必重建或重跑。
- **相同錯誤在 FTS 找到多筆。** 先折疊共享檔案系統的同一份來源。原始訊息、引用錯誤、retry 前後或檢索輸出都可能命中。用可辨認的失敗、根因和驗證結果獨立證實後，才新增 canonical note。
- **想在機群之間同步本機 agent store。** Codex、OpenCode、Cursor 可寫的 runtime state 應留 host-local，共用位置只作 seed。調查用 SQLite FTS5 是可重建本機索引。來源不明的 DB 先隔離，只讀核對 provenance。原始訊息、private browser context、憑證與 agent database 請留在受控位置。

來源整理與同步見 [共享記憶流程](shared-memory.md)。

[原始診斷與查證紀錄](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/global/workflows/agent-session-inventory.md)
