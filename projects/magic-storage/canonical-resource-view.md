---
title: "Canonical key 與 alias 必須分類一致"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: 6df239461d65c23b811106380e22692eb9d6168fddcae5a8521d143e88d060c1
---

# Canonical key 與 alias 必須分類一致

歷史唯讀審查指出，registry 接受 magic_storage:chemical，但 TerminalResourceView 只把 mekanism:chemical 分到 Gas。前者被歸入 Other，而 Mekanism-only 環境又隱藏 Other 按鈕，於是出現「可寫入、不可見」的資源。

對 registry alias 與 canonical ID，從接受輸入、分類到可見分頁逐層核對同一資源的語意。回歸案例須同時覆蓋兩種 ID，並確認已接受的 key 至少能在一個可用 view 中顯示；不能只測 alias 的正常路徑。

來源是帶程式位置的歷史審查報告，未執行完整 GameTest、Gradle 或 GUI；本次也未重現該版本。這裡保留邏輯漏洞與應驗證的契約，不宣稱目前仍有缺陷或已修復。
