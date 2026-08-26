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
- 此 fleet 在 Temurin 21.0.10+7 執行 bundled native solvers 時會在 host
  `libstdc++.so.6` 產生已知 SIGSEGV。Component evidence 使用
  `VGUIDE_SKIP_BROKEN_NATIVE_SOLVERS=1 ant unit-tests`，但這是環境 exclusion，不是
  canonical full command 的替代品。
- 看到 fresh checkout 先因 prefs 缺檔失敗時，恢復 prefs 後要預期 ECJ 可能揭露先前
  被遮住的 dead imports、unused parameters、hiding 或 invalid Javadoc；應清掉
  diagnostics，不要弱化 gate。

# `configuration-checks` Z3 SIGSEGV 的 JDK 邊界

在 mazu Ubuntu 26.04 上，Temurin 21.0.10+7 搭配 repo 原始 JavaSMT 6.0.0 / bundled
Z3 4.15.4，會固定在 `ConfigurationFileChecks -> PolicyIterationManager ->
com.microsoft.z3.Native.INTERNALsolverCheck` crash。`hs_err` 的頂層 native frames 是
`std::codecvt<char16_t, char, __mbstate_t>::do_in` 與 `libstdc++.so.6` null dereference。

已做的隔離實驗：

- `LD_DEBUG=libs` 證明實際載入 repo 的 `libz3.so` 與 `libz3java.so`，不是 Ubuntu 的
  system Z3 4.13.3；runtime files 與 Ivy cache 4.15.4 artifact SHA-256 相同。
- JavaSMT upstream 現用的 Z3 4.17.0-1009b13 在 Temurin 21.0.10+7 仍以相同 stack
  crash；升級 solver 不是修復。
- 只把 `libstdc++.so.6.0.35` 換成 Ubuntu 24.04 的 6.0.33，Temurin 下仍 crash；不能
  歸因於單一 libstdc++ 版本。
- 保持 host glibc 2.43、libstdc++ 6.0.35 與原始 Z3 4.15.4，只切換至
  `/usr/lib/jvm/java-21-openjdk-amd64`（Ubuntu OpenJDK 21.0.11），裸
  `ant -q configuration-checks` 通過（26 秒）。
- Ubuntu 24.04 container（glibc 2.39、libstdc++ 6.0.33、Ubuntu OpenJDK 21.0.12）
  也跑完 3880 configuration cases、無 JVM crash；其普通 failures 只因診斷 container
  未提供 VGuide provider credential，與 Z3 無關。

本機需要完整 native configuration gate 時，顯式使用：

```bash
env JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64 \
  PATH=/usr/lib/jvm/java-21-openjdk-amd64/bin:$PATH \
  ant -q configuration-checks
```

精確責任歸屬仍是「Temurin 21.0.10 與此 Z3 JNI workload 的 runtime interaction」；
沒有同 host 的 Temurin 21.0.11 artifact，尚不能區分 vendor 差異與 21.0.10→21.0.11
patch-level 修復，也不能排除 Z3 undefined behavior 只在特定 JVM memory layout 被觸發。
