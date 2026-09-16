---
title: "失敗數浮動不能直接歸因主機負載"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: d61295ff93df0d6a27f1472d755b578e7cb84ccc50175171868119e6204072df
---

# 失敗數浮動不能直接歸因主機負載

歷史助手看到測試失敗數反覆變動，先稱純負載 flake；後續檢查卻發現 macOS 上的 agy 測試少了舊版的 Linux-only skip guard，轉而追查何時被移除。

結果不穩定只支持待查假設。先核對測試適用平台、原本的 guard 與版本差異，再判定是否為負載或 timeout；只恢復原本有依據的平台限制，不能藉新增 skip 隱藏產品回歸。來源未提供 guard 還原後的測試结果。
