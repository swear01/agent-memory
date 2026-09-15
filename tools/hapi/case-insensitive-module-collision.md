---
title: "僅大小寫不同的模組名稱會遮蔽實際修正"
scope: "tools/hapi"
status: active
updated: 2026-09-15
evidence_digest: 8475aff7a46ab46d0f725cb08f8d6aaa07ddab61deb0e2cca6b9da5e3bc51262
---

# 僅大小寫不同的模組名稱會遮蔽實際修正

歷史 HAPI 記錄指出，StorageUsagePie.tsx 元件與 storageUsagePie.ts helper 在 macOS 開發環境發生解析碰撞；來源回報 TS2724、TS1149 與測試失敗。後續修正摘要將 helper 改名為 storageUsageSlices.ts，連同測試檔與引用一起修改。

新增或改名模組時，檢查大小寫折疊後的 basename 與不同副檔名，包含測試檔。若 helper 與元件撞名，消除檔名衝突並更新引用；不要只靠顯式副檔名或編譯器選項留下碰撞。Linux CI 通過不能代替目標檔案系統上的解析驗證。

來源回報 macOS typecheck 與相關測試通過，也提到完整套件仍有既有失敗；本次未重跑該版本，不宣稱整套測試全綠或歷史 PR 已合併。
