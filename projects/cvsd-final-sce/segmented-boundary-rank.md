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
