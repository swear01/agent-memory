---
title: 實驗室採購各電商平台的發票取得方式
scope: domains/documents
status: verified
created: 2026-10-19
updated: 2026-10-19
tags:
  - procurement
  - invoice
  - yahoo-shopping
  - rakuten
  - autobuy
  - apple
---

實驗室採購資料夾在 Google Drive 本機同步的 `工作紀錄/碩一/實驗室行政/2026-10採購/`（`~/Library` 底下，用 `find ~` 排除 `Library` 會找不到）。`待補發票.md` 記錄還缺的發票、已保存但未寄出的檔案，以及已寄給行政助理的檔案；檔名慣例為 `平台-品項.pdf`。

## Yahoo 購物中心

- 訂單列表：`tw.buy.yahoo.com/usertool/qryprocess.asp?hpp=qryprocess`（`/myaccount/order` 是 404）。每筆訂單有「發票」連結到 `/morder/invoice/home?orderid=<付款單號>&property=SHOPPING`，顯示發票號碼、抬頭與日期。
- 「列印公司戶電子發票」實際請求 `/morder/invoice/print?orderid=<付款單號>&property=SHOPPING`，回傳 `Content-Disposition: attachment` 的 PDF。內建瀏覽器 pane 不存下載（請求顯示 `ERR_ABORTED`），可在已登入頁面用 `fetch(..., {credentials:'include'})` 取 bytes。
- **第一次列印**的證明聯標題是「電子發票證明聯」；之後每次列印都變成「電子發票證明聯(補印)」。報帳偏好無「補印」版本時，要保留第一次下載的檔案。
- 出貨後次日才能列印公司戶發票。

## Apple

- Retail for Business 出貨通知的 `AFE…` 單號不是另一筆訂單；發票 PDF 的「銷售訂單」欄就是 `AFE…`，與 `W…` 網路訂單是同一筆。
- 電子發票由 Apple 台灣分公司另寄郵件，附 `MD<參考號碼>.PDF`。

## 樂天市場（良興等店家）

- 訂單列表 `www.rakuten.com.tw/order/list/`；明細 `/order/detail/?indicationOrderNumber=<訂單號>`，頁面 JS 渲染完才有「發票資訊」：類別、發票狀態（未開立／已開立）、統編、三聯電子檔寄送信箱。
- 店家 30 天內未出貨訂單會自動取消。

## AUTOBUY

- 登入需帳號密碼與圖形驗證碼，agent 不能代登入，需使用者自行下載。
- 下載的 `<發票號碼>.pdf` 是「AUTOBUY購物中心購買證明」，下半部附電子發票證明聯（含買方統編），可直接當發票檔。
