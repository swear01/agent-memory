---
title: "交接最新程式前確認對方看得到分支 tip"
scope: tools/git
status: active
updated: 2026-09-16
evidence_digest: 975cee529fa607f6daa40d9d539418bd62bc3f8018682d43081dc1cd0c08210c
---

# 交接最新程式前確認對方看得到分支 tip

使用者指出關鍵分支有多個本地 commit，遠端卻沒有該分支，要求先 push，否則外部審閱讀不到所稱的最新程式。

把本地完成、遠端可見與審阅所用 commit 分開核對。依當次授權推送確切分支，或提供完整可追溯 diff；不能只給 repo 名稱就假設對方讀到本地改動。來源沒有推送完成或外部審閱結果。
