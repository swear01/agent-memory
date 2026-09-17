---
title: Prism Launcher 更新 CurseForge 整合包丟失模組的成因、復原與設定保留
scope: tools/prismlauncher
tool: prismlauncher
machine: swop
status: verified
created: 2026-09-17
updated: 2026-09-17
---

# 問題現象

在 Windows 11 上的 Prism Launcher (11.1.0) 對 CurseForge 整合包（以 Vault Hunters Third Edition - Remastered 為例）執行「更新整合包 (Update Pack)」後，啟動遊戲直接於 pre-loading 階段崩潰：

```text
ModLoadingException: Mod highlighter requires iceberg 1.0.30 or above
Currently, iceberg is not installed
Caused by: java.lang.NoClassDefFoundError: top/theillusivec4/curios/api/type/capability/ICurioItem
```

檢查 `minecraft/mods` 目錄發現原本 160 個模組全被清空，只剩下整合包 overrides 自帶的 3 個 jar（`buildscape`、`the_vault`、`Highlighter`）。隨後在 Prism 介面重複點擊更新，啟動器也僅會比對版本差分，不會補下載消失的 157 個相依模組。

# 根因分析

1. **CurseForge API 第三方下載限制**：CurseForge 平台部分模組作者關閉了第三方啟動器自動下載授權（Blocked Mods），需要手動下載。
2. **Prism Launcher 更新生命週期陷阱**：
   - 啟動器比對新舊版本時，會先排程將舊版本的既有模組移至刪除清單。
   - 接著嘗試連網抓取新模組；此階段若遇到 blocked mods 彈窗被跳過、網路逾時或中斷，啟動器僅會將新版 zip 內的 `overrides` 目錄解壓進去。
   - 結果造成舊模組全遭抹除，而新模組完全沒有下載到位。
3. **設定覆蓋問題**：解壓 `overrides` 時可能同步覆蓋 `config/`，且修復模組後首次開機時 Forge 初始化會重新生成預設設定檔，若無預先備份會丟失使用者的自訂參數。

# 復原與設定保留 SOP

### 1. 立即完整備份實例目錄
在任何修復前，先複製整套 `<instance-dir>`（例如 `<instance-dir>_backup_settings`），保留 `instance.cfg`、`config/`、`vault-settings/`、`defaultconfigs/` 與 `fancymenu_data/`。

### 2. 萃取官方模組清單與下載來源
從 `<instance-dir>/flame/manifest.json` 可確認整合包宣告的完整檔案數量（如 160 個 `projectID`/`fileID`）。
Prism Launcher 先前抓取中斷的 session 記錄於 `<appdata>/PrismLauncher/logs/PrismLauncher-1.log`，裡面已完整記錄每個模組的直連 CDN 網址：

```text
Will download "https://edge.forgecdn.net/files/<part1>/<part2>/<filename>" to ".../minecraft/mods/<filename>"
```

* **CurseForge Edge CDN 規則**：`fileID = part1 * 1000 + part2`（例：`files/4035/917/` 對應 fileID `4035917`）。直接透過 HTTP GET 下載無需 API Token 亦無頻寬限制。
* **Modrinth 替代來源**：少數模組（如 `BuildersDelight`）若轉移至 Modrinth，可直接取用日誌中的 `cdn.modrinth.com` 網址。
* **Blocked Mods 提取**：使用者手動下載至 `<downloads>` 的檔案（如 `neoncraft2-2.2.jar`）直接移入 `minecraft/mods`。

### 3. 多執行緒並行補下載
透過 Python `ThreadPoolExecutor` 批次連線下載剩餘 150+ 個缺漏的 `.jar` 檔案至 `minecraft/mods`，並校驗檔案大小大於 0。

### 4. 設定與 Config 完整還原
待模組補齊後，將備份目錄中的 `config/`、`vault-settings/`、`instance.cfg` 強制覆蓋回實例，防止初次啟動生成的預設值洗掉個人設定。

### 5. 驗證
比對 `minecraft/mods` 的 `.jar` 數量與 `flame/manifest.json`、`modlist.html` 宣告的數量是否 1:1 精確吻合。
