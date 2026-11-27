---
title: "CVSD SCE left-skip veto 與 argmin prefix"
scope: "projects/cvsd-final-sce"
status: active
updated: 2026-11-27
---

- r24-06 的 SCE left-skip 先算 `lk_n`，右邊界 veto 之後以 `lkN` 決定實際掃描。掃描、`bm` 與 metadata 的載入以及預測比較位元 `bms_q` 都須用同一個 `lkN`。若全掃還載入舊 prefix，舊成本比所有新候選都低時，traceback 會從過期 endpoint 開始，即使當前 DP 各 lane 與各 chunk 最小值全部正確。
- 定向 witness 為 tail seed 7 的第 19 條 seam（p18、width 170）。舊 prefix 為 6293／col 2，正確唯一最小值為 6300／col 21。先對照 `bt_done` 前後的 `lk_n / bclN / lkN / bm / bms_q`，再核對 `upd == (gm && ycv < bm)`。19、32 條都要核對座標和輸出，因為後續刪除會掩蓋大量中間差異。重播工具與數值見 SCE repo `exp/area/seam-argmin/`。
- 另有獨立的 non-WRP left-skip 排程反例。首個真 rotate（`si=1, t=4, ar=1`）時 `hs[1]` 已到 row 2，`hs[2]` 仍是 row 1，`thn` 使 row 2 用到舊門檻。tail31 的 +SEAM=7 和 tail35 的 +SEAM=10 各有一個錯字，到 32 條都恢復 golden。用短 count、逐 seam 代價、threshold 及 compaction 探針驗證早期 left-skip，重播與原始證據見 `exp/area/seam-argmin/tail31-35.md`。此項是 row 2 的 threshold 問題，與 seed7 的 argmin prefix 路徑獨立。
