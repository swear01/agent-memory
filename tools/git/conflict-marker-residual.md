---
title: "合併收尾要檢查殘留衝突標記"
scope: tools/git
status: active
updated: 2026-09-16
evidence_digest: 7e2eb00c2f87dbbcf099af20e1fa7b43f3e8bf26f68854b9b9e845996ab2ba00
---

# 合併收尾要檢查殘留衝突標記

使用者貼出 README 仍含衝突結束標記的行。這只證明當時檔案有殘留，不支持草稿推測的先前人工 review 過程。

合併或 rebase 後掃描受影響檔案的衝突標記，並確認命中是否為刻意引用或未清完的衝突。文字 patch 成功不等於合併內容可交付；來源沒有標記清除後的驗證結果。
