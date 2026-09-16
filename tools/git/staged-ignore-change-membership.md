---
title: "刪除產物的 commit 不會自動包含未 stage 的 ignore 規則"
scope: tools/git
status: active
updated: 2026-09-16
evidence_digest: 45c1cadff1b3df7b283da82dff7dc5150b841d17c88f608c0c0378e1072bca00
---

# 刪除產物的 commit 不會自動包含未 stage 的 ignore 規則

助手用 git rm 加 commit 清理產物，卻漏 stage .gitignore；使用者貼出的未提交 diff 與後续 commit/push 輸出支持這次遺漏和補交。

檢查實際 staged diff 是否同時包含本次產物清理與相應防護，並在 commit 後核對 tree。只提交已核對的路徑，不為補漏而掃入無關 dirty files。

原草稿對 git add -u 的說法不採用；它可以 stage 已追蹤 .gitignore 的修改。這次問題是原操作根本未 stage 那份修改。
