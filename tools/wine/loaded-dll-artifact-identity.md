---
title: "DLL 已部署不代表測試載入了新版"
scope: tools/wine
status: active
updated: 2026-09-16
evidence_digest: d0d2b7f526f2789833048510d95500512f66840fa2c208c42dc5c8db3ae5c2f5
---

# DLL 已部署不代表測試載入了新版

歷史助手回報已把重建的 DLL 放入測試環境，但實際 Wine 載入的仍是 prefix 內舊 system32 下的 dxgi.dll，日誌繼續顯示 Not implemented。當時將問題定位為測試載入位置與預期不同，並表示已移到實際載入位置重新測試。

驗證 DLL 修補前，先以載入日誌或等效證據核對實際載入路徑、產物身分及覆寫規則。複製成功不是執行新版的證據；舊產物產生的錯誤也不能用來判定新修補無效。

來源沒有後續重測結果，故不宣稱修補已生效。此案例不建立 Wine 在所有 prefix 或覆寫設定下的固定搜尋順序。
