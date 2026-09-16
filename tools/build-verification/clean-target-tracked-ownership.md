---
title: "執行 make clean 前辨識會刪到的 tracked 檔"
scope: tools/build-verification
status: active
updated: 2026-09-16
evidence_digest: af5925568d83a9e8d8a379fdbc915be5f6433b885043e0deec7d17be92edc681
---

# 執行 make clean 前辨識會刪到的 tracked 檔

來源回報 make clean 意外刪除團隊追蹤的 include/core/atpg.cpp 與 pattern 檔，先恢復後再查 install rule；保留規則只複製標頭，而該 cpp 與真正 source 不同且未找到 include 使用。

清理前核對 target、追蹤狀態與生成來源，意外刪除先依原內容恢復。後續若懷疑孤兒檔，另查建置與消費者；檔名位於 include 或沒被 include 都不能單獨證明可刪。來源未證明 pattern 沒有 oracle／fixture 用途，也未顯示孤兒 cpp 最後已刪除。
