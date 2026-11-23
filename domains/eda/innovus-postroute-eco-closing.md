---
title: Closing the last few ps of post-route setup in Innovus — curated ecoChangeCell beats broad upsizing or repeated optDesign
scope: domains/eda
tool: innovus
status: active
updated: 2026-11-24
---

# Innovus post-route 收最後幾十 ps setup

CVSD 2026 期末 SCE（tsmc13，28 顆 SRAM macro，1.80 ns，margin-0 floorplan，密度約 91%）實測：

- **同一個 DB 反覆 `optDesign -postRoute` 會越跑越差**：從 −0.022 ns 的 DB 再跑，結果是 −0.05、−0.17、−0.36、−1.83，甚至 −9.7。高密度下 optDesign 的 upsize/buffer 會擠壞 placement 與繞線。
- **大範圍 upsizing ECO 會發散**：把整批違規 path 上的 cell 一起放大，setup 從約 −0.1 惡化到 −0.30／−0.67。原因是輸入電容變大，把上游的弱 driver 拖慢。
- **有效的作法是只換關鍵的 launch flop**：只把關鍵 path 起點的 DFFQ 換成 DFFH 類（clk→Q 較快），setup 收到 −0.022，hold、DRC、connectivity、antenna 與 post-sim 都乾淨。
- **剩下的幾十 ps 用手挑的 ECO 清單**：從 timing report 挑出單級特別慢的 XL gate 與 clk→Q 慢的 flop，逐顆執行 `ecoChangeCell -inst I -cell C`，接著 `refinePlace -eco true` 與 `ecoRoute`，再跑 `ecoRoute -fix_drc`、addFiller、檢查與輸出。每顆都 `catch` 並印出 rc，cell 名稱打錯才看得到。
- **換 netlist 要重跑 APR**：在同一個 floorplan 上換成另一版綜合的 netlist，結果是 −0.24 setup，post-sim 有上千個 timing violation。
- **Innovus 的 setup 與 post-sim 可以不一致**：setup −0.022 的 run，post-sim（native SDF）仍是 0 violation。但交付規則若要求 setup ≥ 0，post-sim 乾淨不能代替 setup。

相關：[[innovus-physical-check-points]]
