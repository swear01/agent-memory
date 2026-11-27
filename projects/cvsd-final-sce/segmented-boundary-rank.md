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
