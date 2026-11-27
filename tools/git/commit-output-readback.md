---
title: Git 提交回讀與審閱方實際可見的分支 tip
scope: tools/git
status: active
updated: 2026-09-11
---

# 可疑 commit 輸出需要 Git 狀態回讀

歷史 CPAchecker 對話中，使用者貼回 Agent 的進度敘述並追問是否完成。敘述先宣稱三次 commit，後又承認 HEAD 未前進、改動仍在 working tree；其中將差異歸因於 shell stdout 污染及 heredoc 問題，但所附資料沒有獨立證明這個根因。

遇到輸出互相矛盾、缺失或可疑 hash，不把成功字樣當作完成證據。用可靠的執行／讀取路徑核對 Git HEAD、commit 物件與內容、working tree 狀態，再判斷是否需要重試。不要只憑前次輸出盲目重做 commit。

原文提出寫 message 檔再用 `git commit -F`，但沒有附上成功結果；這也不能證明所有 stdout 問題都由 heredoc 引起。若當前工具的結果仍不可信，保留改動並明確報告狀態未確認。

## 對外交接時再核對遠端

本地 commit、遠端可見性與審閱方所用 SHA 分別核對。分支只有本機時，依授權推送確切分支，或交付可追溯的 diff。先讀回遠端 tip 與審閱目標，別只提供 repo 名稱。歷史案例只查到最新 commits 未出現在遠端，後續推送與審閱結果未驗證。

[提交案例](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/git/commit-output-readback.md) · [交接案例](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/git/reviewer-visible-tip.md)

Carry 或 cherry-pick 後出現 `nothing to commit` 時，核對指定基底與預期檔案的實際 tree、index 和工作樹。它可能表示變更已等價存在、套用沒有生效或尚未 stage。缺少差異不授權 reset、覆寫或宣稱整批完成。[carry 原案](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/git/nothing-to-commit-content-check.md)
