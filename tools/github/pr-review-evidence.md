---
title: GitHub PR review 的 head、scope 與 OR 證據
scope: tools/github
status: verified
updated: 2026-09-08
---

Swear CheckRun SUCCESS 不一定代表檔案被審查。CPAchecker PR214 的 Markdown-only
變更明確顯示 `Review skipped: no items were selected`；0 findings 是 scope-skipped，
不能稱為實質文件 review。另有 exact-head Gemini 明確無待修意見，才作為該 PR 的
外部審查通過證據；其文件 diff/link/status 驗證另外記錄。

依使用者既定 OR 規則，current-head 的一個 clean provider 足夠。PR211 的
Swear 是 infrastructure/model failure，main 無 protection、rulesets 或 required
checks；GitHub 整體顯示 UNSTABLE 僅來自非必要 provider check。核對最新 head、
MERGEABLE、全部實際 required checks 和 Gemini 的 exact-head clean response 後，
使用普通 `gh pr merge --match-head-commit`，不用 admin，也不改 repository gates。
不能把 optional provider failure 改寫成 CI 全綠。

同一帳號在不同 HAPI session 的 GitHub API capabilities 可能不同。PR214 worker
可 push branch 卻收到 CreatePullRequest permission failure；integration session
以既有授權建立相同 branch 的 ready PR 成功。這證明當次操作權限不同，不證明
GitHub 帳號整體失去權限。可由已授權的 integration session 接手具體 PR 操作，
不用移交 token、修改全域 credentials 或重新向使用者要求已給過的同意。


GitHub review inline comment 的 `commit_id` 可隨新 head 重新定位，不能單靠它判定
finding 屬於最新 review。PR213 的舊 b81 finding 後來顯示 current c27 commit_id；
`original_commit_id` 和 `pull_request_review_id` 仍指向原始審查。核對 review 本身的
head、provider terminal response 與舊 finding 修正狀態。Codex summary 的 Completed
也不等於 clean；有明確「Didn't find any major issues」和相符 reviewed commit 才接受，
不能在它仍帶 findings 時以完成狀態當通過。
