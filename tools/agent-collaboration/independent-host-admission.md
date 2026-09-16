---
title: "獨立主機的工作不能被另一台負載一起擋住"
scope: tools/agent-collaboration
status: active
updated: 2026-09-16
evidence_digest: 83fec859f4ddf7ad9fef723301210824c69a10fc07847829254c8b7f5415dec3
---

# 獨立主機的工作不能被另一台負載一起擋住

歷史助手回報正式 benchmark 為零，等待一台忙碌主機，卻同時列出另一台 idle_ready；使用者明確指出三台獨立，要求使用空閒主機。

對確實獨立的工作逐主機判斷 admission 與派工；有共用瓶頸或同步契約時才採相應共同 gate。監控正常不代表工作開始，也不能僅由零工作數判定排程必定有 bug。來源沒有後續改派或新結果。
