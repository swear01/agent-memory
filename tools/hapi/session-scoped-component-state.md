---
title: "Route 換 session 時元件狀態不能寫到新 key"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: 49fa2064644dcbaf03f73f265bfc6c5194efe49de761605e099293a8e2317746
---

# Route 換 session 時元件狀態不能寫到新 key

歷史 review 指出 DirectoryTree 元件被重用，lazy initializer 只在掛載時讀取 expanded 狀態；切換 session 後可能把舊狀態寫到新 session 的 storage key。來源包含加入 sessionId key 的 commit 與後续 no-issues 回覆。

讓狀態生命週期符合 session 身分，採重新掛載或明確載入新狀態，並測跨 session 切換後保存的隔離性。Review 接受修法不等於此切換已有自動覆蓋；摘錄明確留下測試缺口，本次未重跑。
