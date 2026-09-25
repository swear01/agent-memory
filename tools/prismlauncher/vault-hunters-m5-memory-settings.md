---
title: M5 MacBook Air 上 Vault Hunters Remastered 的 Prism 記憶體設定
scope: tools/prismlauncher
tool: prismlauncher
status: settings-verified
created: 2026-09-26
updated: 2026-09-26
---

# Vault Hunters Remastered 客戶端設定

2026-09-26 在 24 GB Apple M5 MacBook Air、Prism Launcher 11.0.3 上，針對 `Vault Hunters Third Edition - Remastered` 2.0.3（Minecraft 1.18.2、Forge 40.3.11）調整。該實例使用原生 arm64 Microsoft OpenJDK 17.0.15。這是客戶端設定，與伺服器 2.0.4 的維運設定分開。

- 原本 `instance.cfg` 為 `MinMemAlloc=512`、`MaxMemAlloc=4096`、`OverrideMemory=false`；舊啟動紀錄實際帶入 `-Xms512m -Xmx4096m`。
- 現在 `<prism-data>/instances/Vault Hunters Third Edition - Remastered/instance.cfg` 為 `MinMemAlloc=2048`、`MaxMemAlloc=8192`、`OverrideMemory=true`；重新啟動後讀回一致，實際 Java 程序帶入 `-Xms2048m -Xmx8192m`。
- `<instance>/minecraft/options.txt` 的 `simulationDistance` 從 12 降為 8，`renderDistance` 保持 8。調整時已確認遊戲未在執行，以免結束時覆寫 `options.txt`。
- Java 17 在這組參數下自動啟用 G1；`JvmArgs` 保持空白，沒有添加額外 GC 旗標。Prism 曾回報 Forge component metadata 載入失敗，但之後 Java 遊戲程序仍啟動，不能把該訊息記成最終啟動失敗。

8 GB 是堆上限，不代表固定佔用；實際用量可能高於原本的 4 GB 上限。此設定旨在給大型整合包更多堆空間並降低單人世界的區塊負載。**未在世界內量測記憶體曲線、GC 停頓或幀時間，改善幅度未驗證。** 若需要進一步調整，先取得遊玩中的量測資料；不要僅憑設定值宣稱記憶體佔用或卡頓已下降。

參考：[Vault Hunters 安裝指引](https://wiki.vaulthunters.gg/Modpack_Install)、[Prism Java 設定](https://prismlauncher.org/wiki/help-pages/java-settings/)。
