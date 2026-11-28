---
title: CVSD SCE 固定分段的相鄰列 rank 與舊像素版本
scope: projects/cvsd-final-sce
status: active
updated: 2026-11-28
tags: [sce, seam, segment, sram, traceback]
---

固定原始欄位分成八個段時，七個段界在每列目前的 rank 是界左存活像素數。一條合法 seam 的相鄰欄至多差一，任一固定段界的相鄰 rank 因此始終至多差一。差 +1 要增到 +2，較低界左必須刪點且較高界左不能刪點，兩個 seam 欄就至少差二。負差同理。

- 用第一列七個刪除累計（7×5 bits）和後 127 列七個兩 bit 三元 rank 差保存分段資訊，總共 1,813 個 payload bits，另外計 cursor、解碼和更新。只有前 31 次持久寫回才可用 5-bit 累計，第 32 條須只在輸出略過。若也持久寫回第 32 次，基準值 32 要擴位或另編碼。
- 下一列同段的前列 U/M 對齊至多移一欄，parent 還需要兩側最多兩個邏輯 halo。最近的實體鄰段可能已空，應測跳到第二段。
- 跨 seam 共用 pixels 時，下一個 replay 帶在共用界列至多用三個舊 U bytes，包含可能被前帶刪掉的中心。hit W 前將這些舊值明確保留到下一帶。只有 command 無同拍衝突的檢查，抓不到晚發的舊版讀。每筆 Q 需核對版本與地址，並禁止從已覆寫 loc 再發舊 R。

PAT2／PAT5 各重數 28,448 個差值，seed41、全零空段和最左平手亦通過。原工作及重跑腳本見 [SCE-41](https://github.com/swear01/cvsd-final-sce/blob/d15f9f8ec08f823ac44802340473a66dca5ade6d/exp/area/segmented-overlap/sce41.md)。

- 兩個 admitted scan rows 要分配可重用的空 slot ID，並保留各 slot 的絕對 row。兩個 active rows 可以同 parity，且 t+2 Q 歸屬需要用完整地址核版本。量本地 scan 讀寫要看真正接受的 DP vector、halo、hit-W 來源及舊帶移交。SRAM bank 單埠並不限制 local scan 埠數，SCE-51 曾在同一 bank 同拍看到三個 resident 讀與兩個新寫，五址互異。即使 slot 容量足夠也要付這些埠，或用實際仲裁停拍重驗。採 first-hole 搬出 rank low byte 時，把歸還到最後使用的持有狀態、版本和移交都計回。[SCE-51](https://github.com/swear01/cvsd-final-sce/blob/83d3d869c6335779aea914f39e98e811d4c2e4c6/exp/area/segmented-overlap/sce51-two-row-budget.md)。

- 撤掉整帶舊像素並嚴格分相時，先用舊列 issue 到 path release 的最短遞推，再加上必須重讀的存活 bytes／bank 和 I/O。這個界不需要實作仲裁。固定 K13 的 old feedback 加嚴格下一拍取用為四拍；即使新讀打滿 24 banks、跨列／帶理想合併且寫回免費，PAT2＋PAT5 仍需至少 110,388 拍。這只淘汰不允許新讀預取或共用舊像素的嚴格串化，與單純關閉後帶 lookahead 不同。[SCE-56](https://github.com/swear01/cvsd-final-sce/blob/342176bb4cf3a4b4d9cb027a9bfa453ef5c5b5f5/exp/area/segmented-overlap/sce56-band-memory-tradeoff.md)。

- 改用固定 (row-slot id, segment) 槽前，先由逐拍 live masks 建每 bank 的成對衝突圖。跨拍的 clique 決定固定色數。SCE-51 原日曆的 PAT5 單張就使 20 個 bank 為 K16、4 個為 K14，共需 376 槽，超過動態 174 槽。扣四個 tag bits 後，僅固定暫存已超 3×8 的 0.92 cap。全部 376 keys 都有 t+2 Q 與舊帶移交兩種寫入來源。[SCE-59](https://github.com/swear01/cvsd-final-sce/blob/0e180c5f5dd53c82a6e182e059b4757da090d7f8/exp/area/segmented-overlap/sce59-static-bank-slots.md)。

- 從 19-bank 依 row 順序存放的 packed records 切換 24-bank raw 時，依 source 與別列 destination 的實體相交建相依圖，挑沒有其他 live source 會被覆寫的 row，完整解碼進 scratch 再回寫。97 個 188B records 已能擊破逆向逐列，raw row96 先寫會覆寫 row77 的 72 個 source bytes。SCE-79 將 188B scratch、128B lengths、128B indegrees 放在 512 個 raw holes，依真 SRAM Q 重建 metadata；native integrated regression 同時驗凍結後的 held word、實際 t+2、全部 seams/輸出。由同一 native trace 判定拍數。[SCE-79](https://github.com/swear01/cvsd-final-sce/blob/9ac8384180892ac85e21fce962d8602363f91a9f/exp/area/sce79-19plus5/README.md)。

- AS-07 K8 non-WB 的 Q 預解碼先鎖實際 accepted bank/word，再保存該拍 offer 的目標至 t+2。十圖無 late claim，拓撲仍須付資料路。原日曆固定四組六-bank 接收池，即使免費去重和 same-edge bypass，也須各自留 52/44/43/47 B，超同 CTS payload 額度。改單 context 時，先算每帶 n 列的 3n＋ceil(n/4) 下界，因舊 activation 早於當拍 traceback retire。這兩個界各自可先於 DC 排除該具體切面。[SCE-85](https://github.com/swear01/cvsd-final-sce/blob/627829494e79b70a52470167e16bb555ddddc15b/exp/area/sce85-q-landing-budget/README.md)。

- AS-07 K8 兩 active contexts 的實際接收 模式 即使每個六-bank grant 各自落一個 row owner，一個 t+2 edge 仍可能同時要兩個 context 的新 Q 並讀舊 resident。固定池要在同一日曆按組各留最大 occupancy，單共享 aligner 的拆批數要先按同拍必用 bytes 計。SCE-87 的 ctx=2 距首道餘 395 拍，凍住原批次的任意六-byte 單接收至少增 768 拍；交錯雙 lane 的拆批算術可過，但實際需付的選擇／對齊先超嚴格 writer 額度。[SCE-87](https://github.com/swear01/cvsd-final-sce/blob/20eb60f407ec95b72bed6ae7abb28bed3ebf0dcb/exp/area/sce87-two-context-q/README.md)。

- 驗 24-bank RAW 原地寫回時，同拍 R/W bank mask 互斥之外，每筆 WB 都核同 pass、同 addr 的舊 R 已到 t+2，退版要等全部讀 age（含物理重讀）、列 owner 與 resident words 清空。p20 的 175→168 首次縮 8→7 words，p27 的 168→161 只縮尾字，最終 x160 仍可能使用。七條 trits 比四條至少多 640 payload bits，還要計 live RAW words、兩列 thresholds 和 Q 到選位移的完整路徑。只有證到實際退役或同時段無共用需求才能從 CTS 基準扣除。[SCE-92](https://github.com/swear01/cvsd-final-sce/blob/64e06126825c7663b6d4479beb619aae5abd7a3a/exp/area/sce92-raw-fixed-wb/README.md)。
