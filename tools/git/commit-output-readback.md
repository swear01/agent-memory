---
title: 可疑 commit 輸出需要 Git 狀態回讀
scope: tools/git
status: active
updated: 2026-09-11
---

# 可疑 commit 輸出需要 Git 狀態回讀

歷史 CPAchecker 對話中，使用者貼回 Agent 的進度敘述並追問是否完成。敘述先宣稱三次 commit，後又承認 HEAD 未前進、改動仍在 working tree；其中將差異歸因於 shell stdout 污染及 heredoc 問題，但所附資料沒有獨立證明這個根因。

遇到輸出互相矛盾、缺失或可疑 hash，不把成功字樣當作完成證據。用可靠的執行／讀取路徑核對 Git HEAD、commit 物件與內容、working tree 狀態，再判斷是否需要重試。不要只憑前次輸出盲目重做 commit。

原文提出寫 message 檔再用 `git commit -F`，但沒有附上成功結果；這也不能證明所有 stdout 問題都由 heredoc 引起。若當前工具的結果仍不可信，保留改動並明確報告狀態未確認。

來源：Issue25 backlog consolidation v1 的負樣本救回案例；controller-publication-v1 的 commit-readback 對照紀錄。這是一筆被初次模型漏選、經正文及四則相鄰訊息核對後保留的教訓，不代表全部負樣本的漏失率。
