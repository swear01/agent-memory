---
title: CULS GPU resub profiling 與開源版基準
scope: projects/ntu-lsv-course
status: verified
updated: 2026-11-09
tags: [CULS, gpuls, resub, nsight, cuda]
---

## 重現、容量與起點

- **混用作者 binary、開源版、CULS-fast 的成果。** 專案入口 `outputs/PROJECT.md`、開源 baseline 在 `outputs/OSS-baseline.md`；外部作者 binary 供重現與 profiling，don't-care 的 SDCCompute／ODCCompute 未開源。舊本機工具鏈 13.3 超 README 支援範圍，用 CUDA 11.8 devel container、GCC 11.4、cmake，`print.cu` 缺 `printf` 宣告可建置傳 `-DCMAKE_CUDA_FLAGS="-include cstdio"`。新 fork 用 `GPULS_CUDA_ARCH` 明確選架構。
- **rw 分塊變小仍 OOM 或卡在邊界。** 用 `last[]` 差分和 prefix sum 量峰值存活 cut，再算尾端 sort 暫存；已驗規則為 40 B/節點＋64 MB。資源不足走 managed spill，先 `cudaMemPoolTrimTo(pool,0)`。部分 prefetch 會把下一個 pass 擠進溢位。先修 `rwAssign` 非合併清零（每 thread 一組 cut 改成每 thread 一個 int4），再量 PCIe。rf 的大 stack limit 延到 ISOP 前、且早於量剩餘記憶體定 `vSubgTable`。
- **rs root 被排除卻讀到前一批 queue，或空 AIG 下一個 thrust 報 invalid device ordinal。** Rs 丟 root 同時清 simSize 和 `vMFFCNumSaved[root]`。空圖或 strash 移除最後 dangling AND 後，0-grid launch 可能延遲拋錯；從 compute-sanitizer 的第一個 API 失敗查，保留 nNodes==0 early return 和正 grid 邊界測試。

## 卡死、效能與等價

- **只在大 SM GPU 卡住或 illegal access。** Release 的 device malloc 可能回 NULL，assert 被關閉時不可解參考。Rf 換預配分段 arena（`CULS_RF_ARENA_MB`）和 overflow 重做。用 line-buffered log 定位最後 pass，真實 device backtrace 從 cuda-gdb 下啟動。Warp 不應自旋等同 warp 中需過 BSYNC 的 lane；同步輪詢須全 warp 參與或讓 ticket 按相依層分配。A100 修正後 395 次零卡死且 md5 同，仍未單獨確認原始卡住的因果來源。
- **相同卡重跑 md5 一樣，換卡就不同。** 查 rev/plan 分界有無繫於 SM 數、occupancy 或 VRAM。Rw 的原 `tdBlocks=2*SM` 會改選擇，換成常數；用 `CULS_FAKE_NSM`／`CULS_FAKE_TOTMB` 跑 64 格跨配置掃描。回歸用 `i5/md5ab.sh OLD NEW OUT`（原 19 電路 × 4 腳本），rebase 要在新 main 重建兩臂。依使用者 11/09 的範圍，小電路跑 CEC／md5，大電路保留 benchmark 不跑抽樣模擬；main 跑得動仍作 md5 對照。CEC 的 UNDECIDED 保留，`&cec -y` 未定時 `&cec -x` 曾證明同一例。
- **以 profile 推判提速，結果 wall time 變差。** 先用 nsys 定位真正大型 launch，再以 ncu `--launch-skip` 選中。細粒度 phase mark 含同步，full `clock()` 是多執行緒 CPU，不代表 latency。同步區間若 GPU 一直在跑，不能算 host 空檔。strash CPU/GPU 分層取決於每層寬度而非節點總數；過深圖的 spin 吞掉收益。依 NPN class 重排 rw slot 曾破壞 locality、讓評估 3.91→7.02 s；用逐格配對檢查 geomean 遮住的退步。
- **用逐 pass 腳本量新版本，早停卻不生效。** 真正 `resyn2rs` handler 加 `CULS_PASSTRACE=1`；別在中間插 time 變成另一個 flow。PR #24 的 rs 門檻 300 ppm，最後一次仍跑，若 b/rw/rf 又刪 ≥6,000 ppm 則重開。40 電路配對總時間 −18%，AND 平均 +0.038%、最大 +0.349%。PR #25 要連續三次 <300 ppm、rw 與 rf 各自計；只停一次會漏 square 的下一個 rwz 高收益。Dirty-window rs 會漏遠方增益，無法靠這輪結果宣稱等價。

[完整原文](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/projects/ntu-lsv-course/culs-gpu-resub-profiling.md)
