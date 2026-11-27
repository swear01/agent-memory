---
title: CPAchecker ECJ 與 fleet verification gate
project: cpachecker
scope: projects/cpachecker
tags: [ant, ecj, verification, native-solvers]
status: active
created: 2026-08-26
updated: 2026-09-08
---

## Build 與 native JVM

- **乾淨 checkout 在 ECJ 啟動前失敗。** `.settings/org.eclipse.jdt.core.prefs` 是 Ant `build-project-ecj` 的 tracked 輸入，strict policy 會將 warning 改成 error。恢復正確 tracked prefs、修診斷；完整 gate 是 `ant all-checks`。舊 baseline 的 Forbidden APIs 要照跑，和未修改 baseline 比對已正規化的 API／class／source multiset，保留真實失敗狀態。舊工單的 finding 數不可搬用。
- **`configuration-checks` 在 bundled Z3 的 `codecvt<char16_t>`／`libstdc++.so.6` SIGSEGV。** mazu 的 Temurin 21.0.10+7 可重現；換 Z3 或單換 libstdc++ 都沒解。用 distro OpenJDK 21，明確設 `JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64` 及其 `bin` 在 PATH；同套 solver 跑通 configuration 和 unit 階段。fork launcher 有 distro 預設，仍尊重明確 `JAVA`。Ubuntu packaged OpenJDK 21 是實測 workaround；JDK vendor／封裝與 patch level 的效果尚未拆開，未證實 Z3 是根因或 21.0.11 已修復。`VGUIDE_SKIP_BROKEN_NATIVE_SOLVERS=1 ant unit-tests` 只可作已重現案例的 component evidence。[JDK-8379560](https://bugs.openjdk.org/browse/JDK-8379560)

- **看到另一個 SIGSEGV 就沿用 Temurin/JDK 理由。** `std::__codecvt_utf8_base<char32_t>::do_unshift` 可在既有 forked tests 的 libmathsat5j.so，JIT `LinearScan::compute_global_live_sets` 另查。Stock `copysome2-1` 曾在 Ubuntu OpenJDK 21.0.12+8／MathSAT 5.6.15 以 `msat::HashMultiSet<...>::begin()` crash、零 provider calls；單題重跑 UNKNOWN/timeout 沒重現，不能回填原 freeze 或歸因並行度。Launcher／hash 完整不表示 task gate 通過；保留原 crash 與分母。

## 隔離與等價驗證

- **自訂 JUnit 啟動後 witness 找不到相對設定，或跑極慢。** 保留 owner 專屬 class/resource 與 sibling layout，明確給 `-Djava.library.path=<own-worktree>/lib/native/x86_64-linux`。NFS watcher 卡在 `FileOutputStream.open0` 可用等價 JUnitCore；`ClassPath.from` 掃 NFS classes 時可封成私用同 bytes JAR，依序放 `<resource-directory>:<private-classes.jar>:<dependencies>`，優先保留 file URL 和相對 include。JAR-only 曾使 WitnessExporter 失敗；先做 PackageSanity、WitnessExporter 及新增測試。
- **JUnitCore 合併 classes 才出現隨機 fixture failures。** Ant `batchtest fork=true` 是每類別 fresh JVM；等價重跑也要保留 assertions、heap、完整 class inventory 和 skips。JUnit runCount 包含 assumptions。保留中斷的原 Ant 輸出；分開報等價子階段的 PASS 和 canonical gate 結果。
- **已有 binary 能啟動，卻還沒編譯這一輪 Java。** 缺 Ant 時沿用舊 classes 可以作基準 smoke，不能驗證新 patch；核對實際建置工具、載入的 class artifact 與工作樹 byte identity，耗時也不能辨新舊。
- **兩個絕對 `scripts/cpa.sh` 路徑竟跑同一套程式。** 每個 arm 明確設 `PATH_TO_CPACHECKER=<arm-worktree>`、JAVA/JAVA_HOME/PATH；先清掉 inherited CLASSPATH、JAVA_VM_ARGUMENTS、JAVA_GC、JAVA_ASSERTIONS 和無關 VGUIDE 設定，再套共同 recipe。Bash 用 `"${!VGUIDE_@}"` 列變數名逐一 unset，別用 filename glob。用 stub 和實際 wrapper argv/env/CWD 驗證身份及失敗傳播。
- **600 CPU／900 wall 誤變成 610 wall，或外層提早回收 inner capture。** `VGUIDE_TIMEOUT_GRACE=300` 是 CPU 上另加的 wall allowance；outer capture 的 `--termination-grace 15` 用來包住內層預設 10 秒收尾。以實際每台 launcher 做 model-free 檢查，包含 nonzero stage 確實阻止後續階段。
- **需復原 2026-09-02 已移除的 dirty worktree。** 先核對原 stash 內容，再在適合的乾淨 checkout 復原；舊清理批次的 7 個 stash SHA、原保留目錄和搬移收據見下方歷史連結。

[完整原文](https://github.com/swear01/agent-memory/blob/06f50e3a42e850b37cffb6b6a4a446ef670e9625/projects/cpachecker/build-verification.md) · [2026-09-02 清理及復原收據](https://github.com/swear01/agent-memory/blob/06f50e3a42e850b37cffb6b6a4a446ef670e9625/projects/cpachecker/worktree-cleanup-2026-09-02.md)

[舊 classes 紀錄](https://github.com/swear01/agent-memory/blob/2505b5e7b1170ecc3a61858e10894b2af847b5b7/projects/cpachecker/compiled-class-identity.md) · [Temurin／Stock crash 對照](https://github.com/swear01/agent-memory/blob/2505b5e7b1170ecc3a61858e10894b2af847b5b7/projects/cpachecker/native-predicate-test-baseline.md)
