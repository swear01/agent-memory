---
title: DVLab Discord Bot 識別設定與資產規範
scope: projects/dvlab-mis
project: dvlab-mis
tags: [discord, bot, assets, branding, browser-automation]
status: active
created: 2026-09-18
updated: 2026-09-18
---

# DVLab Discord Bot 識別設定與資產規範

## 應用程式資訊

- 應用程式識別碼（Application ID）：`1550481869618024468`
- 名稱：`DVLAB Bot`
- 用途定位：臺大電機 DVLab（設計驗證實驗室）內部專用機器人，提供成員 EDA 工具環境查詢、工作站與伺服器狀態通知等 MIS 自動化功能。
- 官方站台與條款網址：服務條款（TOS）與隱私權政策（Privacy Policy）均指向實驗室正式首頁 `https://dvlab.ee.ntu.edu.tw/`。
- 標籤設定（Tags，上限 5 個）：`internal`、`DVLAB`、`NTU`、`eda`、`tools`。

## 視覺資產規範

### App Icon / Bot Avatar
- 規格要求：1024×1024，比例 1:1（PNG / JPG / WEBP）。
- 裁切考量：Discord 在個人名片、伺服器成員名單與訊息中皆以圓形裁切展示頭像。
- 設計重點：官方標誌（晶片、放大鏡與 Bug 晶片檢視圖騰）必須居中，且不透明圖元半徑應控制在畫布的 76% 內，確保圓形裁切時邊緣零截斷（zero-clip margin）。

### Bot Banner（個人資料橫幅）
- 設定入口：位於 Discord Developer Portal 的 `Bot` 分頁。
- 規格要求：最小 680×240，標準比例為 **17:6**（約 2.833:1）。建議採用 3 倍高解析度規格 `2040×720`。
- 遮擋考量：Discord 個人名片介面會在橫幅的左下方覆蓋圓形頭像（覆蓋區域約佔左側寬度 25%、底部高度 40%）。
- 排版原則：主要品牌標誌、文字標題（`Design Verification Lab`）與研究領域標籤應配置於中段偏右或右側，左側保留純色或抽象背景紋理，避免與頭像衝突。

## 瀏覽器自動化填寫要點

- React 控制元件：直接寫入 `input.value` 無法觸發 React 狀態變更；需使用對應 prototype 的 value setter 呼叫，並派發 `input` 與 `change` bubbles 事件。
- 標籤 Chip 輸入：標籤輸入框需透過 `document.execCommand('insertText', false, tag)` 填入文字後，派發 keyCode 13（Enter）的 `keydown` 事件以建立標籤 chip。
- 圖片上傳流程：將圖片 Blob 注入 `input[type=file]` 並觸發 `change` 後，Discord 介面會彈出圖片裁切對話框（`[role=dialog]`）；必須透過程式點擊對話框內的 `Apply` 按鈕確認，才會正式寫入待儲存狀態並顯示底部的 `Save Changes` 列。
