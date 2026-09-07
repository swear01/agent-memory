---
title: CPAchecker ECJ 與 fleet verification gate
project: cpachecker
scope: projects/cpachecker
tags: [ant, ecj, verification, native-solvers]
status: active
created: 2026-08-26
updated: 2026-09-08
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
  `libstdc++.so.6` 產生已知 SIGSEGV。仍能重現 #30/#111 的 host/JDK 可使用
  `VGUIDE_SKIP_BROKEN_NATIVE_SOLVERS=1 ant unit-tests` 作 component evidence，但這是環境
  exclusion，不是 canonical full command 的替代品。Ubuntu packaged OpenJDK 21 已能跑完
  bare repo-wide JUnit；不要在未重現 native failure 時預設 exclusion。
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

# Fork 已採用的預設與 full-gate 證據

PR #159 將 Ubuntu packaged OpenJDK 21 policy 合併到 fork `main`（merge commit
`40dd75e569e2cf3ad43f38e6e717a47cb4e11e68`）：

- `bin/cpachecker` 未設定 `JAVA` 時，先找
  `/usr/lib/jvm/java-21-openjdk-*/bin/java`，支援 amd64/arm64；明確 `JAVA` override 與
  找不到 distro path 時的原 `java` fallback 均保留。
- Fresh task worktree 在 Ubuntu OpenJDK 21.0.11 執行 canonical `ant all-checks`：strict
  ECJ 編譯 3,480 sources；bare JUnit 4,318 tests、0 failures/errors、734 skips；
  configuration checks 3,880 cases、0 failures/errors、744 skips，沒有 Z3/JVM crash。
- 同一 full command 最後只在 fork `origin/main` 的既有 78 個 VGuide Forbidden APIs
  findings 停止；PR #159 改動 0 Java/VGuide files，沒有跳過或弱化該 gate。
- Latest head `9df14ed827940dc645449dabac68cfe8d29db00f` 的 architecture finding 修正後，
  Gemini review 明確回報沒有 review comments；repository 沒有額外 required CI check、
  branch protection 或 ruleset。

目前可確定的 workaround 是使用 Ubuntu packaged OpenJDK；不要把它誤寫成「Z3 4.15.4
太舊」或「Ubuntu 21.0.11 已修好」。沒有同 host 的 Temurin 21.0.11 artifact，不能把
vendor/packaging 與 21.0.10→21.0.11 patch level 完全拆開；而 JDK-8379560 仍 unresolved。

# Fork baseline 的 ownership 邊界

使用者明確只維護目前 issue/PR 自己變更的部分；未被該 diff 觸及的舊 CPAchecker/fork
baseline code 不應順手併入研究或文件 PR。對已知 78 個 `forbiddenapis` findings，仍要執行並
如實回報 canonical `ant all-checks` 的 baseline failure，但不要關閉或 suppress verifier，也
不要為了讓無關 PR 變綠而修改那些檔案。需要清理時另開明確限定範圍的工作；docs-only diff
使用 diff-local verification，並把 full-gate baseline failure 與本次變更結果分開說明。


# NFS worktree 的獨立 verification runtime

Issue #109 的 bounded audit 證實：只把 classes / Ivy jars 複製至機器本地目錄，會改變
CPAchecker 依 class location 尋找 sibling config、lib、src、test 的根目錄。這會讓既有
TraceFormula、witness、FormulaSlicing tests 出現 missing specification 或 null reached
等假 regression。保留同一 worktree 的 sibling resource layout，並將
`-Djava.library.path=<own-worktree>/lib/native/x86_64-linux` 傳給驗證 JVM；不要借用其他
agent 的 classes 或 rebuild 他人的 runtime。該次所有 11 個失敗類別修正執行環境後重跑
通過，直接使用未修改 baseline classes 的 CPAsTest 也通過。

即使 classes 已本地化，Ant JUnit runner 的 `registerTestCase` 仍會逐案例開啟 worktree
內的 watcher 檔案。thread dump 若顯示 `FileOutputStream.open0`、CPU 時間遠小於 wall
時間，瓶頸可能是 NFS，而非 solver。可用相同 JUnit 類別、classpath、assertions 與 heap
直接執行 JUnitCore，避開 Ant watcher。#109 的 ConfigurationFileChecks 在 Ubuntu
OpenJDK 21、`-ea -Xmx4g` 下直接執行通過 3880 cases；中止的 Ant 記錄必須保留並明確
區分，不能稱為原 Ant command 通過。這不是跳過 ECJ/full-build 規則的普遍授權；只適用
owner 已限定 build 次數、且沿用已完成的獨立 build 進行等價測試的情境。

## NFS ClassPath 掃描：同 bytes 的 JAR 加 file resources

Issue #183 已完成全量資格驗證：Guava `ClassPath.from` 掃描 NFS classes directory，
使 `PackageSanityTest` 的 3 cases 耗時約 241 秒。把自己 worktree 經 canonical
ECJ + javac 新編譯的 5550 個 class/resource entries 原樣封裝至 private JAR，另將
34 個原始 nonclass resources 解出至自己 worktree 的 sibling directory，並以
`<resource-directory>:<private-classes.jar>:<dependencies>` 排列 classpath，保留
resource 的相對目錄與 sibling config base；合格配置的同組 package tests 約 3.5 秒。

不能只用 JAR：`WitnessExporter` 的 `loadFromResource` 會遇到含相對 `#include` 的
設定，`jar:` URL 沒有原始 file base。JAR-only witness smoke 確實失敗，保留該失敗
log；file resources 優先載入才恢復原有 file URL/include 行為。先做 PackageSanity、
WitnessExporter 及新增測試的等價 smoke，再跑完整 inventory。

這次每個原始 test class 使用 fresh JVM、assertions、`enableExpensiveTests=true`，
不借用共享 classes、不重新編譯、不新增 exclusions。全 202 類別共 4346 entries
通過，0 failures/errors；734 原有 skips = 161 ignored + 573 assumptions，
JUnit `Result.runCount=4185` 已包含後者，不能把 runCount 全稱為實際執行 assertions。
`FloatValueTest` 的 570 entries 保留並跑完約 351 秒。另以 canonical 4 GiB heap
及 assertions 跑完全部 3880 configuration entries，0 failures/errors、768 assumptions。
這些是該快照的驗證結果，不是所有環境的固定 test 數或性能保證。

原始 Ant 只停止 owner 自己的程序並保留 partial reports；strict ECJ/javac 證據完整，
但 interrupted `ant all-checks` 不能標為通過。後續完整 Forbidden APIs 仍有 70 個
既有 findings，與 untouched baseline 的 signature/class/source-line multiset 完全
一致；不可用等價 JUnit/configuration PASS 掩蓋該 baseline gate failure。

證據：Issue #183 comments `5554279495`（資源配置與 smoke）、`5554355576`
（全量 JUnit）、`5554373354`（configuration）、`5554450321`（baseline findings
比對）；PR #194 head `40047d7867eac3fe8a5769d05efc910123f61726`。

## 對齊 Ant 的 per-class JVM 與當次計數（#203）

`build/build-junit.xml` 的 `batchtest fork=true` 讓每個 test class 使用新 JVM。
把多個 classes 一起傳給一次 `JUnitCore` 並不等價：#203 的 55-case 合併
invocation 有 5 個 dumper/accounting setup failures，但相同 classpath 下分別
啟動 `PromptAccountingTest`（3）與 `VGuideAnalysisDumperTest`（4）全部通過，
canonical Ant 也通過。不能逕自診斷成缺少檔案或用不被 Configuration builder
讀取的 `-D` 假修復；保留原始例外，先對齊 process isolation。`frozenDir` 是
`VGuideOptions` 的前綴組合 option，grep 不到完整 `vguide.frozenDir` 字串不代表
選項不存在。

當次 full-gate stdout 有 205 個 unit-class summaries：4361 entries、0 failures/
errors、734 skips；另 3880 configuration entries、0 failures/errors、768 skips。
舊 #183 的4346是另一個 snapshot，不能複製到新報告。此後所有 src tree bytes
相同（Git tree `0c0e435ee263aef42557823970ee29d523c5e586`），所以僅 main 文件／
Python 整合不需要重跑完整 Java suite。

完整 gate 仍因68 Forbidden APIs findings exit1。Root 對當次 stdout 與 accepted
#109 baseline 比較 API/class/source-file multiset（只正規化 source line numbers），
68項完全相同、0新增；不是僅比較總數。相同源碼且完整 tests PASS 可使用
repo 明訂的 scoped baseline policy，但不得稱 `ant all-checks` 全綠。證據在
#203 的 `reports/issue203-integration/canonical-java-test-summaries.json` 與
`canonical-forbidden-comparison.json`，原始 artifacts 留在 sibling experiments。

# 絕對 launcher 路徑仍可能被環境導向另一個 runtime

`<arm-worktree>/scripts/cpa.sh` 會優先採用既有 `PATH_TO_CPACHECKER`，最後
exec 該變數所指的 `bin/cpachecker`。所以 control/treatment 即使用不同絕對
launcher 路徑，也可能一起誤跑 parent 的 runtime。每個 slot 必須明確設定
`PATH_TO_CPACHECKER=<that-arm-worktree>`，再搭配 JAVA/JAVA_HOME/PATH；只檢查
命令列上的 cpa.sh 路徑不夠。

Issue216 已以兩個暫存 stub executables 驗證：同一個絕對 cpa.sh 在 inherited
值下選 ambient，明確覆寫後選指定 stub。此檢查沒有啟動 verifier/JVM。
這是既有 launcher 的設定優先序，不需要為實驗改 production launcher。

`bin/cpachecker` 還會把 inherited `CLASSPATH` 放在 arm classes 前面，並採用
`JAVA_VM_ARGUMENTS`、`JAVA_GC`、`JAVA_ASSERTIONS` 等 launcher overrides；隔離實驗的
wrapper 應先清除這些父程序設定，再套用明確的共同 recipe。Bash 的
`unset VGUIDE_*` 只做 filename glob，不能清除同前綴變數；要遍歷
`"${!VGUIDE_@}"` 得到變數名稱後逐一 unset。以無害 sentinel 和 stub 驗證環境清除，
不需要為 wrapper 檢查再呼叫模型或 verifier。
