---
title: "HAPI 封存 Codex 後，原生對話檔可能擋住 reopen"
scope: tools/hapi
status: active
updated: 2026-09-25
---

# Codex archived session 無法 reopen

在 zeus 驗證：HAPI session 封存的同一秒，原 Codex rollout 移到 `<remote-home>/.codex/archived_sessions/`。HAPI `SpawnHappySession` 對 Codex resume 呼叫 `findCodexSessionPath(resumeSessionId)`，但 `getCodexSessionRoots()` 只掃 `<remote-home>/.codex/sessions/`。因此工作目錄存在、runner workspace root 正確，reopen 仍回 `Codex session path is unavailable or outside workspace roots`。這個錯誤表示找不到可解析的原生 Codex thread 路徑；不要誤判為工作目錄不在允許範圍。

來源核對：`tiann/hapi` [PR #847](https://github.com/tiann/hapi/pull/847) 的 `cli/src/api/apiMachine.ts` diff 加入了 Codex resume 路徑檢查。該 PR 已關閉、未合併；2026-09-25 上游 `main` 沒有這段錯誤字串，維護版的 `03-pr-847.patch` 則帶入了檢查。上游 issue #819 談的是較早的 archived session 缺少 Reopen 功能，並非此路徑檢查回歸；搜尋上游及維護版 issue 尚未找到完全相同的回報。

已驗證的單次復原：先確認 archived rollout 的 `session_meta.id`、`cwd`、內容與 HAPI 對話相符，再執行 `codex unarchive <codex-thread-id>`，然後透過 HAPI 對指定 session reopen／送診斷訊息。原 session 成功變成 `active: true`，並回覆診斷訊息。再次封存後可能重現；HAPI 的掃描範圍仍待修正。
