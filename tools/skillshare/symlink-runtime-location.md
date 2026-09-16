---
title: "Skill 同步後重新確認 runtime 的實際落點"
scope: tools/skillshare
status: active
updated: 2026-09-16
evidence_digest: e5ed45c586f3b165f341ab4756c0c79bef52899fb116194534757aa07da68956
---

# Skill 同步後重新確認 runtime 的實際落點

歷史助手發現 active skill 路徑同步後成為指向 durable source 的符號連結，而預期位置沒有原先共用 venv，因此先檢查路徑並提出恢復 runtime。

同步或切換 skill 安裝方式後，解析 active path 的實際目標，核對 interpreter 與依賴是否仍可用，再決定重用或重建。不能假設原路徑下的 venv 必然跟著搬移；也不將歷史的來源內重建提案變成所有 repo 的 venv 儲存政策。來源沒有重建成功結果。
