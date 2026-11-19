---
title: "XMCL 在 Windows 上的 Forge 啟動問題：重複模組與遊戲路徑編碼"
scope: tools/xmcl
status: active
updated: 2026-11-20
---

# XMCL 在 Windows 上的 Forge 啟動問題

以「神秘启旅 1.10.5」（Minecraft 1.20.1、Forge 47.4.21、XMCL 0.51.5、Java 17）為例。先用 XMCL 安裝日誌確認 Forge 和資源下載完成，有實際啟動紀錄後再讀實例的 `logs/latest.log` 和最新崩潰報告。畫面的「未安裝該版本」可能只是當下仍在安裝。

## 同模組 ID 的 JAR

XMCL 顯示「有 1 個模組重複了」時，讀取 JAR 的 `META-INF/mods.toml` 並比對檔案雜湊。這個版本的下載清單有兩個不同檔名的 `netmusic` JAR，實際內容完全相同。保留 `netmusic-1.5.1-forge+mc1.20.1.jar`，把另一份加上 `.disabled` 後移到 `mods` 外備份。重開 XMCL 可重新讀取清單。重裝或修復可能依原始清單把第二份抓回來。

## 繁中 Windows 上的遊戲路徑

崩潰報告有 `java.nio.file.InvalidPathException: Illegal char <?>`，堆疊指向 `net.brazier_modding.gdarp.CommonClass.<clinit>`，且 `--gameDir` 裡有超過 MS950 字元集範圍的字（例如簡體「启」）。該模組包自帶的修改版 Global Packs 19.3.5 會從 `sun.java.command` 重新解析 `--gameDir`。用相同 Java 17 和路徑實測，啟動參數已將「启」替換成 `?`，所以工作目錄能讀也會崩潰，`-Dfile.encoding=UTF-8` 也修不了該參數。

完全退出 XMCL 後，先備份實例的 `instance.json`，再把受 XMCL 管理的實例資料夾與 JSON 的 `name` 改成同一個純 ASCII 名稱（例如 `Mystic-Journey-1.10.5`）。保留整個實例和 Global Packs。XMCL 0.51.5 會從 `<xmcl-data>/instances/` 重新發現實例，並嘗試讓受管理的資料夾名稱和 `name` 一致。同一層改名後，共用的 libraries、versions 和全域資料包仍可讀，XMCL 也已重新載入實例並顯示 308 個啟用模組。本次驗證到啟動器重新載入，遊戲主選單仍待確認。
