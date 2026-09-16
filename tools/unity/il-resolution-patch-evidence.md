---
title: "解析度 byte 命中必須核對 IL 操作與參數"
scope: tools/unity
status: active
updated: 2026-09-16
evidence_digest: e567b79b9fa259262431c938e13eee117c7775ad6a3b392fb583117beb549cf8
---

# 解析度 byte 命中必須核對 IL 操作與參數

歷史助手指出 1470 的正確 little-endian 搜尋值是 BE 05 00 00，使用者範例卻寫 5E；895 的命中有 branch operand 與非 method body，SetWindowPos 又帶 NOSIZE。

核對數值編碼、指令邊界與真正 API 參數，不把任意 byte 命中當尺寸常數。已檢查呼叫沒有可確認的安全 patch 點，應保留而非猜改；這不能證明所有其他層都無尺寸來源。來源未提供所要求的 boot.config 前後及重啟回寫結果，Wine／client-area 歸因仍是假設。
