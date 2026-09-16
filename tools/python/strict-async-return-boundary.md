---
title: "Strict 失敗不能被內層 return None 吞掉"
scope: tools/python
status: active
updated: 2026-09-16
evidence_digest: dc332c2566f481bf6c0305fa06084935cf541bfd0801b60eed5897b061181224
---

# Strict 失敗不能被內層 return None 吞掉

歷史助手發現非同步 batch 以 None 返回時，外層 except 不會執行，即使 strict 模式也不拋錯；使用者要求類似 fallback 規則涵蓋專案其他路徑。

在共同失敗邊界保留 strict 與允許降級的契約，沿內層返回、gather 結果與外層例外處理核對傳遞。錯誤訊息有印出不能代替失敗狀態。來源包含修改摘要及一次替換字串找不到，沒有完整回歸證據；不能單憑摘要認定所有 fallback 都已修正。
