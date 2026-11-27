---
title: Impeccable hooks 的設定來源、信任與出貨驗證
scope: tools/impeccable
status: active
updated: 2026-10-16
---

- **Skill 讀得到，但 hooks 未載入或未信任。** Skillshare 管 skill 與 target links；來源更新後依其流程同步。transfer_MAC 的 `scripts/sync-ai-agent-configs.py` 從 `stow/agents/.agents/impeccable-hooks.json` 渲染 Claude `settings.json` 和 Codex `hooks.json`。Codex renderer 使用 `~/.agents/skills/impeccable/scripts/impeccable`；launcher 缺檔先核對實際載入來源。只認可已審過的 Impeccable handlers，保留其他設定與既有信任。
- **Hook 動了，不代表 UI 驗收完成。** 以 Codex 的 `skills/list`、`hooks/list` 確認發現及各 handler 的狀態，再用相關 Edit／Stop 實例驗證。後端編輯保持靜默是當時的 smoke 之一。歷史 browser modal probe 逾時，native Windows 尚無實測證據；PR merge 或 CI 也不能代替這兩項。自動 Skillshare audit 有警告，後續人工判讀不能當成 audit 自動通過。
- 按全域 AGENTS.md，交付前對改動 UI 跑 Impeccable distill，移除多餘文案並保留重要操作提示、無障礙與必要警告。只改文字偏好不必重裝 engine 或重設 hook 信任。

[原始部署快照、PR 與驗收範圍](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/impeccable/fleet-installation.md)
