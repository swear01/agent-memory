---
title: "OpenCode effort 選項須沿 ACP 回報同步"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: c388538d610d48f61e86d503b4f51a6f981e5ef77aea2bd741d8208e279a2b44
---

# OpenCode effort 選項須沿 ACP 回報同步

歷史 issue 指出 Web 硬編 effort 列表而 CLI 直接轉送，可能碰到 Effort not found；ACP 已回報 thought_level options，後續 commit 訊息另指出 fallback 成功後 Hub 仍廣告被拒絕的值。

將當次 model／session 的 capability 傳到 UI 並在送出邊界驗證，若政策允許改用其他支援值，須在 ACP 成功後同步實際值，不能維持虛假的選擇。來源有設計、程式片段與 commit 描述，沒有完整端到端證據；不能由 CLOSED 推定所有版本都修好，也不外推到其他 flavor。
