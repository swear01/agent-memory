---
title: Pincremental mobile Gold save schema
scope: project
project: Pincremental
status: active
created: 2026-09-05
updated: 2026-09-05
---

# Pincremental mobile Gold save schema

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

Steam build 已由公開 metadata 確認是 Electron + Greenworks 且含 IAP，但
匿名 SteamCMD 無法取得 depot（`Missing configuration`，需要帳號 license），
因此尚未直接解包 Steam binary。Steam 支援 `dAustin.Gold` 是根據同時代的
跨平台程式結構所做的高信心推論，不應描述成已直接驗證。
