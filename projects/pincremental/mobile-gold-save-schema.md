---
title: Pincremental Steam and mobile save schema
scope: project
project: Pincremental
status: active
created: 2026-09-05
updated: 2026-09-05
---

# Pincremental Steam and mobile save schema

## 已驗證結論

Pincremental Android `1.10.79` 的 bundled JavaScript 仍宣告內部
`GameVersion = 1.107`，並以 client-side 變數 `Gold` 保存 Gold 餘額。這推翻
「Gold 只存在 IAP backend、Export Save 不含餘額」的假設。

APK/XAPK 樣本 SHA-256：
`A9FDB696CFAD2DA3FE6563993F7AA4B2BE4AE6E078BBF6AFC3A48F59C84D8CDC`。

`SaveGame()` 會建立 `dAustin` object，包含：

- `Gold`
- `DonPointMulti`
- `DonTicketMulti`
- `DonHSexp`
- `DonExtraIncome`
- `DonShareMulti`
- `DonLives`
- `WarpSpend`
- `TotalSpend`

接著以 `btoa(JSON.stringify(dAustin))` 寫入 `localStorage.dAustin`。
`ExportSave()` 則序列化整個 `localStorage`、base64 編碼，再依序替換：

```text
Z -> ~
S -> >
B -> Z
M -> S
```

Import 使用相反順序還原，對每個 imported key 直接執行
`localStorage.setItem(key, value)`，再呼叫 `LoadGame()`。`LoadGame()` 讀取
`dAustin.Gold` 時沒有向後端驗證，因此修改 Export Save 裡的
`dAustin.Gold` 可直接改變一般載入後的 Gold。

## 購買與 Restore Purchases 邊界

一般購買完成後，程式從 product identifier（例如 `gold20`）解析數量並執行
`Gold += GoldAmount`。`Restore Purchases` 則遍歷 RevenueCat
`nonSubscriptionTransactions`，加總所有 `gold*` 產品數量，只扣除
`WarpSpend`，最後用計算結果覆寫 `Gold`。

所以修改後的 Gold 會在一般 Save/Load 與 Export/Import 中保留，但按下
`Restore Purchases` 會以真實交易紀錄重算並覆寫它。

## 平台差異與證據邊界

舊 Web/Kongregate 程式只有 Kreds 永久效果欄位，沒有 Gold UI；同一份
Export Save 可以攜帶未知的 `dAustin` key，但網頁版不會顯示或使用 Gold。

Steam build 已從使用者本機安裝的 `app.asar` 直接解包驗證。套件版本為
`1.05.0`，遊戲內部 `GameVersion = 1.107`；程式同樣宣告 `Gold`，並在
`SaveGame()` / `LoadGame()` 透過 `dAustin.Gold` 保存及讀取，沒有後端載入
驗證。因此 Steam 的 Gold client-side save 行為是已直接驗證，不再只是
跨平台程式結構推論。

## Steam Mana prestige eligibility

Steam v1.107 的 `AvailableManaNow()` 只用
`floor(log(TotalInvestors * 5) / log(5))` 計算按鈕文字顯示的 Mana 數量；
顯示例如「Sacrifice for 18 Pinball Mana」不代表按鈕一定可用。

`WizardReady` 還要求所有直接 investor upgrades（排除 `AutoCompanyOn` 和
`PermaBuyers` 容器）以及 `InvUpgrades.PermaBuyers` 內的每個項目都不是
`"no"`，且 `AvailableMana >= 4`。實際故障案例的兩個 blocker 是
`PermaBuyers.AutoShares = "no"` 與 `PermaBuyers.AutoArcade = "no"`；把它們
設成 `"yes"` 後，依原始碼判定 `sacrificeEnabled = true`。這是 UI 的誤導性
差異：標籤只反映 investor 數量，disabled 狀態另受 upgrade completeness
控制。

## Steam 成就與啟動方式

Pincremental 的 Steam App ID 是 `1369470`。實測以檔案路徑直接啟動
`Pincremental.exe` 後執行 Sacrifice，沒有觸發 Steam 成就；關閉遊戲並從
Steam 手動重新啟動後，成就才成功觸發。因此，只要任務目標包含 Steam 成就，
後續操作必須透過 Steam launcher 啟動：

```powershell
Start-Process "steam://rungameid/1369470"
```

或使用已安裝的 `steam.exe -applaunch 1369470`。這會走 Steam 的遊戲啟動
流程並初始化 Steam client／overlay。不要直接執行遊戲 `.exe` 來完成成就
相關操作。存檔修改仍應在遊戲完全關閉時進行，修改完成後再由 Steam 啟動，
最後在遊戲內正常執行會觸發成就的動作；不要只離線改成就結果欄位。

## Import Successful 但欄位沒更新

Web 版 Import 只驗證外層存檔能否解碼；內層 object 若含拼錯或損壞的 key，
仍會顯示 `Import Successful`，而 `LoadGame()` 會靜默忽略未知 key。

一次實際案例中，來源存檔的 `LifetimeMana` 被 Unicode replacement
character 損壞成 `Lifet\uFFFDMana`，因此 `Mana` 與 `TotalMana` 可載入，
但 Lifetime Mana 維持舊值。同份存檔另有三個損壞 key：
`Invest\uFFFDrMulti`、`AutoInvest\uFFFDrs`、
`toggleaut\uFFFDInvestorMulti`。修復時應逐一 decode 所有 base64 內層
JSON，並斷言每個 decoded payload 都不含 U+FFFD，而不是只檢查外層
JSON；外層看到的只是 base64 字串，不會暴露這類損壞。
