---
title: "建置找不到 Java 時先查已安裝 JDK"
scope: tools/java
status: active
updated: 2026-09-20
evidence_digest: e8d6709cb0271602e857e2d439466f12394273db90d946ef0f1f1a28f3d693cc
---

# 建置找不到 Java 時先查已安裝 JDK

歷史助手誤以為需要另裝 Java，後承認本機已有 Homebrew keg-only JDK，只是沒有進入 shell 的 JAVA_HOME/PATH；使用者要求直接完成設定。

先核對已有工具鏈、專案要求與實際 shell 環境，再安裝或更換。keg-only 不等於未安裝，JRE 也不等於具備建置用 JDK。來源包含助手自述使用既有 JDK compileJava 成功，但未附可獨立核對輸出，也未證明永久 shell 設定完成。

## 重複 JDK 清理：mazu / athena，2026-09-20 實測

「不是目前預設的 Java」不足以證明某套 JDK 可以刪除。先查套件、實際
`java` / `javac` 路徑、`JAVA_HOME`、腳本與符號連結；NFS home 還必須查
共用該目錄的其他主機，避免只看本機程序就刪掉共用工具鏈。

- mazu 已安裝系統 OpenJDK JDK / JRE 21.0.12，位置為
  `/usr/lib/jvm/java-21-openjdk-amd64`；athena 的同一路徑也已實際執行
  `java -version` 與 `javac -version` 確認正常。
- mazu 的互動環境 `java` / `javac` 仍使用 `<remote-home>/.local/jdk-21`
  的 Temurin 21.0.10；`/usr/bin/java` / `javac` 則指向系統 OpenJDK
  25.0.4。系統已安裝版本、shell 預設與 alternatives 預設必須分開查。
- 額外的 `<remote-home>/jdk21` 安裝已刪除，釋放約 346 MiB；沒有移除
  系統 Java 21，也沒有變更 shell 預設。刪除前查 mazu / athena 可讀取的
  程序 executable、cwd、memory maps 與 Java 環境，未發現引用；少數
  systemd、PAM、SSH 程序的 environment 無法讀取，不視為完整環境稽核。

清理時發現本機 CPAchecker 舊版
`scripts/vguided-cegar/launch_isolated_run.sh` 的 build fallback 仍引用
`$HOME/jdk21`。已改成系統 OpenJDK 21 路徑，保留顯式 `JAVA_HOME` override，
同步更新該 checkout 的 `docs/notes.md`。`JAVA_HOME` 未設定、空值與指定
另一套 JDK 三種情境，皆使用腳本的實際 build command 完成 Ant 編譯及
執行 Java 21 測試程式；另通過 `bash -n` 與 `git diff --check`。這不代表
重新執行 CPAchecker 全套測試或正式實驗。

修正保存在本機分支 `chore/local-jdk21-cleanup-20260920`，commit
`f75119cfe0`，並已套用本機原 checkout；沒有推送遠端。遠端主線早在
`8ce4edd683` 已刪除該舊腳本，因此沒有為此重加腳本或建立 PR。回放更舊
版本前仍需確認 JDK 路徑，不能假設歷史 commit 已包含這次本機修正。
