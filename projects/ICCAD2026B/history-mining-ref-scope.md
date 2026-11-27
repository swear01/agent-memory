---
title: "宣稱歷史挖完前核對 refs、拓撲與 pathspec"
scope: projects/ICCAD2026B
status: active
updated: 2026-09-16
evidence_digest: 40d2273ad24e174c08c9c0e7f4322d196d3db2345dc3d7c4eda51baa67d25d1e
---

# 宣稱歷史挖完前核對 refs、拓撲與 pathspec

使用者質疑候選太少；歷史助手才發現先前只挖 pin 的祖先，物件庫其他 refs 尚有較新的歷史，並比較窄路徑與全路徑的 commit 數。

記錄實際 refs、祖先範圍、路徑與重新命名邊界，再描述涵蓋範圍。現在的 tree 沒子目錄不能證明歷史路徑都相同；更大的 git log 數也不等於全是可用或獨立的新候選。來源沒有後续探勘完成證據。

要擴充 PR 衍生 bug list 時，先凍結 PR 來源、分類與 confirmed 條件。以 mux／driver 枚舉合成注入、靜態候選數或相近故障風格，均不足以證明歷史來源或 confirmed 產量。

[合併來源](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/projects/ICCAD2026B/pr-derived-corpus-scope.md)
