---
title: Cypher Assist 截圖與解謎驗證
scope: projects/cypher
status: active
updated: 2026-11-24
tags:
  - cypher
  - cypher-assist
  - braille
  - playfair
  - linear-b
---

# Cypher Assist 截圖與解謎驗證

## 如何讀取玩家目前看到的題目？

Windows 的 Game Patches 資料夾有 `Cypher/assist.ps1`。已載入 Cypher Assist 的遊戲可以從遊戲內產生 PNG。在 Game Patches 根目錄執行：

```powershell
$cypherCapture = Join-Path $env:TEMP ('cypher-' + [Guid]::NewGuid().ToString('N') + '.png')
& './Cypher/assist.ps1' capture -Output $cypherCapture
```

讀取工具回傳的 PNG 路徑。這次用同一指令多次取得當前畫面，能在不操作視角、終端或答案的情況下讀題。畫面角度改變時重新定位圖形，座標不可沿用。使用者要一起推導時，以畫面和使用者提供的參考資料為準，按要求停在辨識、假設或逐格對照的階段。查找符號表時，搜尋範圍維持在文字系統的對照資料。

## 如何驗證符號辨識或解碼？

- **點字。** 一格有兩欄三列。遊戲會把未啟用的點位置略為移開，放大原圖並比較同一橫排的點中心。視角帶斜率或左右區段高度不同時，按當前截圖的局部基準線判定。先用短行能否讀出完整提示來驗證選點規則。
- **Playfair。** 先用已知的明文與密文組合反推 5×5 方陣，再逐組重新加密驗證。循環平移整張表不改變映射。未出現在已知對照的字母可能互換，候選方陣也可能有其他差異。把候選方陣都套用到目標密文，再檢查各自的明文。
- **Linear B。** 同時使用 Unicode 的 Syllabary 和 Ideograms 兩張表。有些遊戲符號混用音節、表意符號與疊字。先逐格記下讀音、英文名稱或能描出的字母，再驗證連讀規則。Windows 若已安裝 Noto Sans Historical，可用它在本機渲染兩個 Unicode 區段的 PNG 對照表，並先用字型 cmap 核對字元覆蓋。

完整解碼字串可能仍是另一個提示。把逐字轉錄、語句解讀和遊戲判定分開回報。曾檢查到 Challenge 終端比對大寫化後的字元與精確長度，作答時以連續字母為準。保留畫面中實際讀到的尾碼，並區分可驗證的解碼與對填充的推測。
