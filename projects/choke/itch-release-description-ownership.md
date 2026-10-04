---
title: choke itch.io 發布與手動 description 保護
scope: projects/choke
status: verified
updated: "2026-10-04T15:58:33+08:00"
---

## 使用者維護的頁面內容

- 使用者在 2026-10-04 明確要求：itch.io 的 `description` 已由本人手動編輯，agent 不得再修改或覆蓋。後續建置、上傳與切換遊戲檔案的授權不包含修改 description。
- 本次上傳／切換播放版本時多次儲存了整個 Edit project 表單，description 也一起提交；使用者回報這覆蓋了手動修改。不能把舊表單內的 description 當成最新內容，也不能宣稱已還原使用者的手動版本。
- 後續只處理已授權的建置與遊戲檔案；避免儲存整張編輯表單。優先採用確認只更新遊戲檔案的發布途徑，不順手更新說明、操作文字、圖片、credits 或 tags。若切換版本必須提交整張表單，先停下該提交，不能以先重新載入表單為由保證沒有覆蓋風險。
- 只有使用者再次明確授權某項頁面內容修改，才能處理該項；本次沒有重新寫回或還原 description。

## 已驗證的建置教訓

- 此次 WebGL 增量建置的 BuildReport 顯示 Succeeded、0 errors，公開頁仍發生 `Cannot create required material because shader is null` 和 `Cannot initialize a LocalKeyword with a null Shader.`；建置成功不等於公開版可玩。
- 對照 packedAssets，故障包缺少 `CoreCopy.shader`、`CoreBlit.shader`、`CoreBlitColorAndDepth.shader`。對 URP 設定執行 ForceUpdate／ForceSynchronousImport，再以 `CleanBuildCache` 完整重建後，三個 shader 都已入包，瀏覽器錯誤消失。這是此次已驗證的修復方式，尚不能把 importer／增量快取的精確因果當成通用結論。
- 僅還原本次建置自動修改的 URP／PlayerSettings 與產生的測試 metadata，保留隊友變更；還原設定後需重新匯入，避免下一次建置使用舊 importer 狀態。
- 2026-10-04 發布快照：來源 main `def599904cc5b93b5384f509e8ce0cdc98cc7468`；PlatformAirflowLabTests 6/6 通過，539 個輸入檔 hash 建置前後一致；只打包 `Assets/Scenes/PlatformAirflowLab.unity`，含正式 Opening logo 與兩張 comic；修正版 itch upload `19552701`。公開頁實測開場、R、Space、氧氣死亡過場與自動重生，重生時氧氣 98%、分數 0，當輪瀏覽器 errors 0。這是當時快照，之後發布仍須重新驗證。
- SharedArrayBuffer 支援當時維持關閉：這個建置 threadsSupport=false；只勾 itch.io 選項不會使遊戲獲得多執行緒效能。
