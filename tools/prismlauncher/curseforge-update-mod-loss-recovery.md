---
title: Prism Launcher 更新 CurseForge 整合包丟失模組的成因、復原與按鍵設定保留
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

檢查 `minecraft/mods` 目錄發現原本 160 個模組全被清空，只剩下整合包 overrides 自帶的 3 個 jar（`buildscape`、`the_vault`、`Highlighter`）。隨後在 Prism 介面重複點擊更新，啟動器也僅會比對版本差分，不會補下載消失的 157 個相依模組。此外，更新或重建後，玩家自訂的按鍵綁定（如衝刺、丟棄等）與伺服器列表全數消失。

# 根因分析

1. **CurseForge API 第三方下載限制**：CurseForge 平台部分模組作者關閉了第三方啟動器自動下載授權（Blocked Mods），需要手動下載。
2. **Prism Launcher 更新生命週期陷阱**：
   - 啟動器比對新舊版本時，會先排程將舊版本的既有模組移至刪除清單。
   - 接著嘗試連網抓取新模組；此階段若遇到 blocked mods 彈窗被跳過、網路逾時或中斷，啟動器僅會將新版 zip 內的 `overrides` 目錄解壓進去。
   - 結果造成舊模組全遭抹除，而新模組完全沒有下載到位。
3. **按鍵與客戶端設定的儲存位置**：
   - 模組參數保存在 `config/`，但玩家的所有**自訂按鍵綁定（Keybindings）、GUI 縮放、視野 (FOV) 與音量**全保存在根目錄的 `options.txt`，伺服器列表則在 `servers.dat`，小地圖路標在 `xaero/`。
   - 更新或刪除實例時若未特別備份根目錄檔案，這些設定會被一併刪除。
4. **遊戲進程退出時的覆寫競爭（Write-back Race Condition）**：
   - Minecraft 在啟動時將 `options.txt` 讀入記憶體，並在**關閉遊戲（進程退出）的瞬間將記憶體當前值覆寫回 `options.txt`**。
   - 若在遊戲運行期間手動將舊的 `options.txt` 放回，當遊戲關閉時，進程會直接用記憶體中的預設值重新覆蓋檔案，導致還原完全失效。

# 復原與設定保留 SOP

### 1. 立即完整備份實例目錄
在任何修復前，先複製整套 `<instance-dir>`（例如 `<instance-dir>_backup_settings`），保留 `instance.cfg`、`config/`、`vault-settings/`、`defaultconfigs/` 與 `fancymenu_data/`。

### 2. 從 Windows 回收筒救援原始按鍵與地圖資料
若更新或刪除時遺失了 `options.txt`、`servers.dat` 或 `xaero/`：
- 檢查對應磁碟的 `$Recycle.Bin`（例如 `E:\$Recycle.Bin`）。
- 搜尋含有 `options.txt`、`servers.dat` 與 `vault_soundOptions.txt` 的回收資料夾。
- 先將回收筒中的整個舊實例暫存至獨立目錄（如 `<instance-dir>_original_recovered`）備用。

### 3. 萃取官方模組清單與下載來源
從 `<instance-dir>/flame/manifest.json` 可確認整合包宣告的完整檔案數量（如 160 個 `projectID`/`fileID`）。
Prism Launcher 先前抓取中斷的 session 記錄於 `<appdata>/PrismLauncher/logs/PrismLauncher-1.log`，裡面已完整記錄每個模組的直連 CDN 網址：

```text
Will download "https://edge.forgecdn.net/files/<part1>/<part2>/<filename>" to ".../minecraft/mods/<filename>"
```

* **CurseForge Edge CDN 規則**：`fileID = part1 * 1000 + part2`（例：`files/4035/917/` 對應 fileID `4035917`）。直接透過 HTTP GET 下載無需 API Token 亦無頻寬限制。
* **Modrinth 替代來源**：少數模組（如 `BuildersDelight`）若轉移至 Modrinth，可直接取用日誌中的 `cdn.modrinth.com` 網址。
* **Blocked Mods 提取**：使用者手動下載至 `<downloads>` 的檔案（如 `neoncraft2-2.2.jar`）直接移入 `minecraft/mods`。

### 4. 多執行緒並行補下載模組
透過 Python `ThreadPoolExecutor` 批次連線下載剩餘 150+ 個缺漏的 `.jar` 檔案至 `minecraft/mods`，並校驗檔案大小大於 0。

### 5. 避開進程鎖定，還原 Config 與 options.txt
- **必須確認 Minecraft (`javaw.exe`) 完全關閉**才進行檔案覆蓋，或啟動背景監聽守護行程：
  ```powershell
  $proc = Get-Process -Name javaw -ErrorAction SilentlyContinue
  if ($proc) { $proc.WaitForExit() }
  Start-Sleep -Seconds 1
  Copy-Item "<recovered>/options.txt" "<instance>/minecraft/options.txt" -Force
  Copy-Item "<recovered>/servers.dat" "<instance>/minecraft/servers.dat" -Force
  Copy-Item "<recovered>/config/*" "<instance>/minecraft/config" -Recurse -Force
  Copy-Item "<recovered>/xaero/*" "<instance>/minecraft/xaero" -Recurse -Force
  ```
- 待進程完全退出後自動重新覆蓋，即可徹底避開 Minecraft shutdown 時的預設值寫回。

### 6. 驗證
1. 比對 `minecraft/mods` 的 `.jar` 數量與 `manifest.json` 是否 1:1 吻合。
2. 啟動遊戲確認自訂按鍵與伺服器列表完整保留。
