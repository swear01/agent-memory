---
title: "退出全螢幕需要確認先前確實進入過"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: 5b2f0cd341615b07feede5ca4700cdd2f2f7b8eb9712d0e0b652ad2399f4f8f8
---

# 退出全螢幕需要確認先前確實進入過

歷史助手依 crash report 將問題縮到 setMode(monitor=0)：建立初始一般視窗也經過此路徑，redirect 卻將它當成退出自製全螢幕，改動 Cocoa 視窗屬性。

以每個視窗的實際狀態轉移判斷退出條件，不能只用與初始化共用的參數值。測試初始視窗與真正進入再退出兩條路徑，避免未取得狀態就執行回復操作。來源只有診斷與回歸測試計畫，沒有 crash 消失的證據。
