---
title: NFS checkout 的 Git push 可用本機 bare transport 保留原 commit
scope: tools/git
tool: git
machine: valkyrie
status: verified
updated: 2026-09-06
---

NFS shared checkout 的普通 `git push` 曾長時間停在 `D` / `rpc_wait`；改用同一
common Git directory 加 invocation-only `-c core.bare=true` 後，`send-pack` 仍等待
NFS。當時 `git cat-file` 讀取新 commit 正常。未完整定位 NFS 根因，不能把這個
現象當成 CPU 不足或測試失敗。

已驗證 workaround：確認並停止自己卡住的 push process tree，在本機檔案系統
建立獨立、無 checkout 的 bare transport，使用既有 object store 作 alternates，
直接推送完整 tested SHA 到指定 branch。這不重建 commit、不改主 checkout，也
不更動全域 Git 設定。

```bash
git init --bare "<local-transport>"
printf '%s\n' "<common-git-dir>/objects" > "<local-transport>/objects/info/alternates"
git --git-dir="<local-transport>" push "<remote-url>" "<tested-sha>:refs/heads/<task-branch>"
```

完成後獨立查詢 remote ref，確認它等於原 tested SHA，再建立 PR。不要在 push
仍 pending、remote branch 尚未存在時提前建 PR。HAPI 任務須以真實 session job
包住 push；不要用假的 sleep meter。

此方法仍需要原 object store 可讀；若 object reads 本身也停住，不能保證有效。
不要刪除別人的 lock、修改 shared classes、reset dirty checkout 或改 remote URL
來繞過問題。transport 不會自動設定原 task branch 的 tracking ref，後續可在 NFS
正常後用一般 fetch/branch tracking 操作補齊。

驗證來源：CPAchecker issue 179 / PR 188；local 與 remote commit 都是
`016680216f67cba4fd1837966d7529205721231f`。
