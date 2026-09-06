---
title: Google Photos 全量 HEIC 工作的範圍與瀏覽器規則
scope: domains/media
status: active
updated: 2026-09-06
tags: [Google Photos, Takeout, HEIC, Motion Photo, in-app-browser]
---

# Google Photos 全量 HEIC 工作的範圍與瀏覽器規則

- 使用者要求全部照片轉 HEIC 與清理時，Takeout 必須選 Google Photos 的全部年份、全部相簿。不得因磁碟空間不足或想先試做，就自行只匯出某一年；抽樣驗證不能取代完整工作範圍。
- 空間限制只影響下載、解壓、轉檔及驗證的批次大小。全量匯出可使用 2 GB ZIP 分卷，後續逐卷處理並追蹤全部進度。
- 使用者明確要求使用 Codex 內建瀏覽器，不占用其滑鼠。此工作使用 in-app browser 的頁面操作；不得自行切回原生桌面控制或 Brave 前景操作。
- Takeout 相簿控制項可能延遲載入。提交前等待並確認「已包含所有相簿」，必要時開內容選項核對所有年度均勾選；提交後確認「Google 正在建立」的成功狀態。
- 2026-09-06 的錯誤是只選 2024 年。更正後已於當日 21:53（Asia/Taipei）在內建瀏覽器建立全部相簿匯出。這只證明匯出已建立，不代表下載、HEIC 轉換、回傳或清理已完成；後續須重新確認即時狀態。
- Android Motion Photo 規格支援 HEIC，但普通靜態圖轉碼會遺失動態部分；須保留影片、聲音與 XMP 並正確封裝。Apple Live Photo 是照片與配對影片。不能把動態照片一律判定不能轉，也不能未經 Google Photos 回傳播放驗證就刪除原件。
- 靜態照片既定試轉品質為 80（有損），保留拍攝時間、位置等資料。原件與 Takeout JSON 保留到回傳驗證完成；清理必須精確對應且確認占用容量，不能依年份推定免費與否；不自動清空垃圾桶。
- 大型影片轉存工作中的「照片不處理」只適用該影片流程，不得套用來否定後來已授權的全量照片 HEIC 工作。
