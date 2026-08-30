---
title: CPAchecker VGuide 實驗 provenance 與 preregistration 陷阱
project: swear01/cpachecker
tags: [vguide, experiments, provenance, preregistration]
status: active
created: 2026-08-30
updated: 2026-08-30
---

# Frozen benchmark pairing

`docs/vguided-cegar/benchmark_sets/loops_reachsafety_unreach.list` 的 SHA-256 `c072daf0...6ae` 與 SV-Benchmarks commit `2e1723fde6aa65a250dcb677efa45edaa4b6b631` 是同一組 frozen input。不要因較新的 checkout 是 clean 就混用；`9cf91981...d` 缺少其中 26 組 source/task 路徑。Formal run 前要同時驗證 commit、clean tree、manifest hash、764 組 source/task 全數存在。

# Issue #166 G0 邊界

完整 764-task source-only census 的 deterministic diagnostic 是 6 個 structural matches，其中 5 個已在 exposure denylist，只有 `c/loop-invgen/heapsort.yml` 這 1 個 unexposed candidate。預先規定的 continuation threshold 是至少 24 個、涵蓋至少 3 個 strata，因此 #166 在 G0 停止，沒有 Stock gate、Muse generation 或 consumer replay。這不能推論 prompt 泛化成功或失敗；#149 的已接受單案例結果也不受影響。

該 census 的 GitHub preregistration comment 晚於 run 完成一秒，正式證據標為 procedural-invalid，只保留 deterministic diagnostic。不要用看過結果後的重跑假裝恢復 blind preregistration。

# 安全發布 preregistration comment

GitHub comment 若含 Markdown backticks、`$()` 或 launch command，不得插值進 shell command。Backticks 會被 shell 當成 command substitution，可能在 comment 建立前直接啟動實驗。先把 body 以正常檔案編輯流程建立，再執行：

```bash
gh issue comment ISSUE --repo OWNER/REPO --body-file COMMENT.md
```

Launch 前再讀回遠端 comment，確認 body hash、內容與 timestamp；remote comment 尚未存在就不得啟動 formal job。
