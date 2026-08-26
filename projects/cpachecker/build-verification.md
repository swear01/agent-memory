---
title: CPAchecker ECJ 與 fleet verification gate
project: cpachecker
tags: [ant, ecj, verification, native-solvers]
status: active
created: 2026-08-26
updated: 2026-08-26
---

# ECJ prefs 是 build input

`.settings/org.eclipse.jdt.core.prefs` 不是可刪的 IDE-only state。Ant target
`build-project-ecj` 會複製它、把其中所有 `warning` 轉成 `error`，再以 ECJ 編譯全部
Java sources。Commit `141939b173` 誤刪此檔，造成 fresh checkout 在 ECJ 啟動前失敗。

Issue #154 / PR #156 以刪除前的精確 blob
`389f0ebe6b55817e8d14c6ab649869b004c7804d` 恢復 tracked prefs，並清掉重新啟用 gate
後揭露的 28 個既有 diagnostics；沒有降低 compiler policy。Merge commit：
`93f02e9f1897e81acb7574cfab8462a965597178`。

# 驗證命令要分層解讀

- Canonical full command：`ant all-checks`。它必須保留 strict ECJ build；不能用跳過 ECJ
  或較弱 prefs 來取得綠燈。
- 此 fleet 的 bundled native solvers 會在 host `libstdc++.so.6` 產生已知 SIGSEGV。
  Component evidence 使用
  `VGUIDE_SKIP_BROKEN_NATIVE_SOLVERS=1 ant unit-tests`，但這是環境 exclusion，不是
  canonical full command 的替代品。
- 看到 fresh checkout 先因 prefs 缺檔失敗時，恢復 prefs 後要預期 ECJ 可能揭露先前
  被遮住的 dead imports、unused parameters、hiding 或 invalid Javadoc；應清掉
  diagnostics，不要弱化 gate。
