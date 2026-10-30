---
title: Verto 輸入容錯（input tolerance）評分工具
scope: projects/gamejam1 (repo swear01/Verto)
status: active
updated: 2026-10-31
tags: [Verto, level-design, difficulty, route, solver, tolerance, dotnet]
---

# Verto 輸入容錯評分

- 工具：`Assets/Phase/Tests/Editor/PhaseTolerance.cs`，用 `dotnet run -c Release --project Verification/Tolerance` 產生 `Verification/ToleranceReport.md`（PR swear01/Verto#49）。文件 `Documentation/Tolerance.md`。
- 分級（60 Hz 幀數視窗）：tight ≤ 3、moderate 4–12（目標帶）、free ≥ 13。關卡判定 in band / too tight / too loose。
- 已驗證的設計重點：
  - 控制器的危險只隨跳躍改變（無時間驅動），所以 A* 合併狀態只需位置、速度、顏色、平台位置等。
  - 搜尋路線常帶空中急停、牆跳等「技巧」，會讓視窗被誤判成 1 幀；需要：方向變換成本、刪除多餘繞路（後續輸入以位置錨定重放）、多組搜尋設定取最寬鬆路線、再加上錄製路線。
  - 單一的「位置錨定」後續輸入模型會誤判緩衝跳（例：level-a 某跳實際 ±3 幀可過卻判 1 幀）；改為「位置錨定或保持節奏」任一能恢復即算成功。
  - Chaos 關量到起跳 ±8 幀，和 `ChaosLevel.md` 宣稱的 ±8 px 一致，可當交叉驗證。
- 效能：Umbrella（924 塊、160 雷射）每幀約 1.3 ms，全報告約 8 分鐘；Gear 的 Z 字形要把右緣落腳點高度放進啟發函數才找得到路。
