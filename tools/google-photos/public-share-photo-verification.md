---
title: Google Photos 公開分享照片的原圖與地點核對
scope: tools
status: active
confidence: high
created: 2026-09-03
updated: 2026-09-03
tags:
  - google-photos
  - shared-album
  - exif
  - geolocation
  - browser-automation
sources:
  - live Google Photos public-share workflow
---

# Google Photos 公開分享照片的原圖與地點核對

## 能力邊界

- 公開 `photos.app.goo.gl` 分享連結只授權查看該分享項目，不能據此搜尋使用者完整的私人 Google Photos 圖庫。
- 若登入流程卡在密碼、驗證或帳號限制，不應把公開分享頁誤當成完整相簿搜尋替代品；應請使用者提供目標照片或相簿的公開分享連結。
- 分享頁顯示的畫面可用來辨識內容，但店名、拍攝時間與地點仍應由原始檔 metadata 或其他獨立證據確認。

## 取得可驗證原圖

1. 開啟使用者提供的公開分享連結，確認分享頁中的項目數與目標照片。
2. 優先使用 Google Photos 的下載功能取得原始檔，不要把頁面上的縮圖或 `lh3.googleusercontent.com` 顯示尺寸當成原圖。
3. 下載後檢查 MIME、影像尺寸與 EXIF。Google Photos 分享頁可能顯示經縮放版本，但下載原始檔仍可保留拍攝時間與 GPS。
4. 將原圖視為暫存證據；若後續只需放進文章，另製作長邊約 2000 px 的 web 版本，避免直接上傳數 MB 原圖。

## 時間與店家地點核對

- 拍攝時間以原始 EXIF 為主要證據；檔名時間只作交叉檢查，不能單獨當結論。
- 有 GPS 時，把照片座標與 Google Maps 店家座標做距離比對。距離很近只能證明照片在店址附近，仍要配合照片內容、使用者敘述與店家名稱，避免把同名店搞混。
- 同一店名若存在不同地址或分日共用店址，必須把完整地址一起核對；不要只靠搜尋結果中的店名合併紀錄。
- 沒有 EXIF／GPS 時，明確標記未驗證，不要從餐點外觀猜出確切店家與日期。

## 隱私與公開文章

- 原始相簿分享連結、EXIF、GPS 與完整檔名可能暴露個人行程；除非文章確實需要且使用者明確要求，否則不要放進公開文章或公開 repository。
- 公開文章只使用重新縮圖後的副本與必要說明；原始照片不提交到 memory repository。
- 不保存 Google 登入資料、驗證碼、cookie、session identifier 或短效圖片 URL。

## 完成條件

- 照片內容、拍攝時間與店家地址的證據來源分開記錄，且沒有把推測寫成已確認。
- 文章用圖片實際可載入，尺寸適合網頁；原始檔與敏感 metadata 未被意外公開。
- 若資料來自公開分享連結，結論只涵蓋該連結內的項目，不聲稱已搜尋完整私人圖庫。
