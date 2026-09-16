---
title: "TeX 百分比漏跳脫後不要把舊 PDF 當新輸出"
scope: tools/documents
status: active
updated: 2026-09-16
evidence_digest: 237172c9347ac4fb54396eb27f61ffc3ad676a8f86191d0bfe0b0ff01e37f81a
---

# TeX 百分比漏跳脫後不要把舊 PDF 當新輸出

來源顯示 build exit 1 與掃描 frame 到檔尾的錯誤，同時仍列出 PDF 頁數。助手指出 note 文字中的數字百分比未跳脫，註解掉同一行結尾，頁數来自舊 PDF。

在一般 TeX 文字參數中正確跳脫百分比，按語法上下文修正，不用全域替換破壞正常註解或 verbatim。建置失敗後既有 PDF 的頁數或可開啟性不能證明新內容成功產出。來源沒有乾淨重編與渲染驗證結果。
