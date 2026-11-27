---
title: AS-07 BT area gate for paid pipeline registers
scope: projects/cvsd-final-sce
status: active
updated: 2026-11-28
---

# AS-07 BT 流水暫存的面積早停

AS-07 的 matched A/B 可能維持相同暫存位數，但原 A 是實驗 fixture，wide104 的 BT 帳本只列已納入的 payload 與 flags。合成前先按 capture edge 對帳。逐 instance 核對原來已計價的角色、存活區間、新角色與可回收條件。Weight、frontier 和下一列可同時存活的 L/R 候選和各有用途。Writer 餘額獨立，retired-cone 抵扣以該次已證的實體 instance 為準。

若有人建議因等寬而略過面積關，先從鎖版 trace 選一個真實雙列 issue，取 valid 幾何及原像素窗。把固定 tap 下的對角和最低位寫成像素 parity 的線性式，對當拍有效 capture 求 GF(2) rank。兩個獨立 14 像素窗的 X/U 關係是路徑圖，選取有效邊後仍可驗每個保留位獨立。按該次 `slow.lib` 的最小 one-bit edge FF 面積計下界，從當前模式預算扣除已承諾項，僅扣有證據的重用。若最低必要未計價面積已超額，離線功能驗證可留證，DC 停於面積關。映射 A/B 差額保留為未測。

專案證據見 SCE-27 delayed carry 的報告與 `area_bound.py`。SCE-23 source tap 另有一例，用完整控制域的互異 one-hot 真值函數數量先界定 edge FF 與 decode 的容量。 [SCE-27](https://github.com/swear01/cvsd-final-sce/blob/d4bd07ce0d1bc15aaf8f10cf8efa629dc41a274c/exp/area/as07-delayed-carry/report.md) [SCE-23](https://github.com/swear01/cvsd-final-sce/blob/7aef4f5e340fd427f0a4bcf15e9c635d6038255b/exp/area/as07-source-tap/report.md)
