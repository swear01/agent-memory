---
title: "阻止檔案 drop 導航時保留非檔案拖放"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: 5085ca425716bf6c61b8ddec001e0e687954c22d7e59887cf3e851c9ef0adfd2
---

# 阻止檔案 drop 導航時保留非檔案拖放

保存 diff 顯示 document 的 drop 原先只清狀態，未阻止檔案預設開啟；另一方面 drop zone 又在辨識檔案前一律 preventDefault，連選取文字拖入 composer 都被取消。

在 dragover 與 drop 分別處理 Files payload 的預設動作；非檔案拖放保留原生行為。附件功能 disabled 時仍阻止檔案導致導航，但不新增附件。

來源含修補 diff、測試自述與 follow-up review，後者確認原拖放 findings 已處理但仍有 locale 問題。本次未重跑瀏覽器驗收，不稱整個 PR 所有問題都完成。
