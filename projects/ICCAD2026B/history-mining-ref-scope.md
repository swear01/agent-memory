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
