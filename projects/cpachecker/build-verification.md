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

網路上的精確對應證據：

- OpenJDK [JDK-8379560](https://bugs.openjdk.org/browse/JDK-8379560) 是 unresolved 的
  21.0.10 Linux bug，native frames 與本案相同：`libstdc++.so.6+0xdfc31`、
  `std::codecvt<char16_t, char, __mbstate_t>::do_in+0x4e`、
  `std::ostream::_M_insert<unsigned long>+0x92`。原始 workload 是 VLCJ，不是 Z3，故此
  crash signature 不專屬於 solver。
- Microsoft OpenJDK [issue #670](https://github.com/microsoft/openjdk/issues/670) 的維護者在
  Microsoft Build、Temurin、Corretto、Zulu 21.0.10 都重現，Ubuntu 套件的 OpenJDK 則
  正常，並據此建立 JDK-8379560。該 issue 後來因 Compose 1.10.2 消除原 workload 的
  crash 而關閉，不代表 JDK bug 已修復。
- JetBrains [CMP-9994](https://youtrack.jetbrains.com/issue/CMP-9994) 的最小 reproducer
  只在 native code 執行 `std::cout << (unsigned long)42`，仍可進入錯誤的
  `codecvt<char16_t>`；它支持 process-global C++ locale/facet state interaction，而非
  Z3 formula 或輸入內容損壞。
- OpenJDK build 文件說 `--with-stdc++lib` 預設採 static linking。ELF `DT_NEEDED`
  本機比對也顯示 Temurin JDK libraries 不依賴動態 `libstdc++.so.6`，Ubuntu OpenJDK 的
  `libjvm.so` 等 libraries 則與 Z3 一樣動態依賴 system libstdc++。因此目前最符合全部
  證據的機制是：static libstdc++ 的 JDK 與 dynamic libstdc++ 的 JNI library 同處一個
  process，造成兩套 C++ locale/facet global state 或 symbol interposition；facet identity
  錯配後，整數 ostream 路徑取到 `codecvt<char16_t>` 並 null-dereference。這仍是推論，
  JDK-8379560 尚未確認最終 root cause。

本機需要完整 native configuration gate 時，顯式使用：

```bash
env JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64 \
  PATH=/usr/lib/jvm/java-21-openjdk-amd64/bin:$PATH \
  ant -q configuration-checks
```

目前可確定的 workaround 是使用 Ubuntu packaged OpenJDK；不要把它誤寫成「Z3 4.15.4
太舊」或「Ubuntu 21.0.11 已修好」。沒有同 host 的 Temurin 21.0.11 artifact，不能把
vendor/packaging 與 21.0.10→21.0.11 patch level 完全拆開；而 JDK-8379560 仍 unresolved。
