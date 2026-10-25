---
title: CULS GPU resub profiling 與開源版基準
scope: projects/ntu-lsv-course
status: verified
updated: 2026-10-25
tags: [CULS, gpuls, resub, nsight, cuda]
---

# 專案位置

本機專案在 `<documents>/CULS-profiling`。入口為 `outputs/PROJECT.md`；`outputs/OSS-baseline.md` 為開源版對照。

2026-10-25 起是 git repo，private GitHub `swear01/CULS-profiling`（已 `gemini-link`）。`.gitignore` 排除所有 Nsight 原始 capture（`*.nsys-rep`、`*.ncu-rep`、`*.sqlite`、`outputs/*-evidence.tar.gz`）與作者 binary／zip；CULS 原始碼修改留在 cthulhu `/var/tmp/culs-oss-20261024/CULS-abl/`，完整 diff 在 repo 的 `work/qor-gap/relax-div.patch`。

量測都在 cthulhu（RTX 4070 Ti 12 GB）上進行，只測過小型 EPFL 電路，尚未重現 TCAD 2026 論文的大電路 speedup，因為需要 ≥24 GB VRAM。

# 原始碼範圍

- 作者寄來的 `gpuls` 只有 binary：CUDA 11.8 建置，含 sm_60／sm_86 與 PTX，沒有 strip。
- 公開 repo `cuhk-eda/CULS`（BSD-3）包含 base GPU resub（DAC 2024）與 strash：`markReadyNodes`、`insertLevelNodes`、`updateLevelNodesNewIds` 在 `src/aig/strash.cu`；`collectDivisor` 與 `resubDivs*` 在 `src/algorithms/`。
- 不開源的只有 don't-care 版本，也就是 `SDCCompute` 與 `ODCCompute`。這部分不建議反編譯：SASS 無法還原成可編譯的原始碼，而且向作者索取 binary 時申明的用途僅限重現結果與 profiling。

# 建置開源版

- README 限定 CUDA 11.4–12.5。cthulhu 主機上只有 CUDA 13.3，所以用 `nvidia/cuda:11.8.0-devel-ubuntu22.04` container 建置，GCC 11.4，container 內需要 `apt-get install cmake`。
- `src/misc/print.cu` 缺少 `#include <cstdio>`，會出現 `identifier "printf" is undefined`。加上 `-DCMAKE_CUDA_FLAGS="-include cstdio"` 即可，不必修改原始碼。

# 開源版 vs 作者 binary（div，`rs -n 2`，2026-10-24）

- 兩者輸出都通過 `cec -n`，彼此也等價。QoR 不同：開源版 and 為 45,226，作者版為 45,280，level 都是 4,405。
- 作者版的輸出結構每次不同，但節點數相同；開源版的輸出逐位元組 deterministic。
- 三個逐層 kernel 在兩版中都跑了 4,405 次，等於電路深度。差別在開源版每層多了 thrust `copy_if`／`remove_if`，kernel 總數 30,916 次、共 68.19 ms；作者版為 13,302 次、41.21 ms。作者版已把壓縮併入 `markReadyNodes`（見下節）。
- conflict resolution 和 cut 計算的 kernel 在兩版中不同。
- 結論：開源版可以當改進研究的 baseline，但只能跟未修改的開源版比較，不能拿作者 binary 的數字當基準。

# strash 預先分層（2026-10-24）

- 修改只在 `src/aig/strash.cu`：host 端以 DFS 算出每個節點的 level，依（level, id）做 counting sort 後一次上傳；逐層迴圈只剩 `insertLevelNodes`／`updateLevelNodesNewIds`，拿掉 `markReadyNodes` 與 thrust `copy_if`／`remove_if`。每層的節點集合與順序和原版相同，所以 id 分配和輸出都不變。
- 坑：resub 之後 AIG 的節點 id **不是**拓撲順序，fanin id 可能大於節點 id。用單次 id 掃描算 level，會產生 ABC 讀取時觸發 `Vec_PtrEntry` assertion 的壞 AIG。
- 結果：div、adder、multiplier 各 6 次輸出都和原版逐位元組相同，也都通過 `cec -n`。div 的 full time 從 0.17 s 降到 0.07 s，`cudaStreamSynchronize` 從 17,658 次降到 37 次，kernel 總和從 68.3 ms 降到 36.5 ms；multiplier（272 層）幾乎沒有差別。報告在 `outputs/strash-levelsort.md`。
- cthulhu 上 nsys 2026.1.3 連續跑兩個 `nsys stats` 報表時，第二個會因為「Existing SQLite export ... older than input file」輸出空白並讓 `set -e` 腳本中止；加 `--force-export=true` 即可。

# 作者 binary 的 PTX 對照（2026-10-24）

- 方法：`cuobjdump -ptx` 抽內嵌 PTX，`nm -C`／`objdump -d -C` 看 host 端（binary 沒 strip）。只做分析，不還原原始碼，不讀 SDC／ODC。報告在 `outputs/author-binary-diff.md`。
- 作者版 `markReadyNodes`（12 參數）沿 fanout CSR（`Aig::getFanoutGPU`）展開上一層 frontier，fanin 都已有新 id 的 fanout 以 `atomicCAS` 佔位、`atomicAdd` 加入下一層。同層順序由 atomic 決定，這就是作者版輸出結構每次不同的原因。
- 作者版每層仍有 3 次 `cudaDeviceSynchronize`、1 次 D2H 讀回大小與 1 次 D2D 搬 frontier，div 上共 13,252 次同步；預先分層版只有 71 次。
- 作者版的 conflict resolution 在 CPU（`calcRemainMFFCGreedy`），開源版預設走 GPU 的 `resolveMffcConflictGreedyGPU`；作者版另有 `getMFFCDrivenReconvCutResub`，以及以 `srand(time(0))` 產生的隨機向量（`rs -n 2` 是否用到未確認）。

# conflict resolution 改在 CPU（2026-10-24）

- 開源版 `resolveMffcConflictGreedyGPU` 以 `<<<1,1>>>` 啟動：greedy 依 gain 逐一 commit，後面的 root 依賴前面改寫的 `vMaskValid`／`solidDiv`，無法平行，單一 GPU thread 跑得比 CPU 慢。開源版本來就有 CPU 實作 `resolveMffcConflictGreedy`，只在候選 ≥300,000 時使用，所以重現作者的做法不必反編譯。
- 在預先分層版上加 `CULS_CONFLICT_CPU` 環境變數切換：div 的 conflict kernel 15.5 ms 消失，phase 3 結束時間從 0.035 降到 0.022 s，full time 0.07 → 0.06 s；div／adder／multiplier 輸出與預先分層版逐位元組相同並通過 `cec -n`。報告在 `outputs/conflict-cpu.md`。
- 未驗證：大電路時 CPU 版每次 `cudaMallocHost` 並來回複製 `nObjs × DECGRAPH_SIZE` 的 `decGraph`，可能反而變慢。

# MFFC-driven cut（自行設計，2026-10-25）

- 在 `getReconvCutResub` 原本的 reconvergence 展開前加第一階段：以和 `visited` 對齊的 `visitedRefs`（初值 `numFanouts - 1`，MFFC 節點展開時遞減 fanin）找出所有 fanout 都在 root MFFC 內的 cut 葉，只展開這類葉；level ≤ root-10、PI、超過 `nMaxCutSize` 不展開。環境變數 `CULS_MFFC_CUT` 切換。是依概念自行設計，沒有從作者 binary 還原。報告在 `outputs/mffc-cut.md`。
- `rs -n 2` 結果（open-source／MFFC／作者）：div 45,226／45,280／45,280；mem_ctrl 46,523／46,511／46,501；adder、multiplier、hyp 與 open-source 相同。15 個輸出全部通過 `cec -n`，hyp 的 CEC 每次約 650 s。候選 root 變多不保證最終較小：greedy conflict resolution 會選出不同集合。
- 坑：開源版在 mem_ctrl 上本來就偶爾 non-deterministic（10 次中 1 次輸出不同、節點數相同），和 conflict 在 CPU／GPU 無關；比較 determinism 時要先跑多次 baseline，不能把差異歸給新 patch。

## ABC `resub` 對照（2026-10-25）

- 等價設定：CULS `rs -n 2` ↔ ABC `resub -K 8 -N 2`；CULS 不保持 level，對應 ABC 加 `-l`。ABC `-F`（ODC）預設 0。
- cthulhu 4070 Ti 5 個電路：CULS resub 本身比 ABC 快約 1.8–3 倍（div 0.07 vs 0.13 s、hyp 0.17 vs 0.30 s），但 CULS process 只讀寫就要 0.19–0.21 s（CUDA context + 2 GB heap），小電路整體 wall 反而輸 ABC；只有 hyp 打平（0.36 vs 0.40 s）。
- QoR：multiplier CULS 26,000 vs ABC 26,249；hyp ABC 207,940 vs CULS 212,475（ABC 好 2.1 %）；mem_ctrl ABC 46,369 vs 46,523；div/adder 幾乎相同。hyp 差距原本推測來自 conflict 丟棄，後來證實不是（見下節）。
- 證據在專案 `outputs/abc-baseline.md`、`work/abc-baseline/`。

## hyp QoR 差距：除數層級限制與 `CULS_RELAX_DIV`（2026-10-25）

- 不是 conflict：連跑 8 次 `rs`，hyp 只到 212,447；CULS 只找到 2,256 個候選，ABC 做了 6,325 次替換。真正原因是 `collectDivisor` 在 `fUseConstr` 下要求 fanout 擴展的除數 `level ≤ level(root)`（同層 id 較小），用意是讓整批平行提交不形成回路；ABC 沒有這條限制。拿掉後 hyp 候選變 7,490、結果 207,225（ABC 207,940）。MFFC 只展開 10 層幾乎無影響，深度設 1000 會觸發 `MAX_MFFC_SIZE = 64` 的 assert。
- 坑：直接拿掉限制會產生回路，mem_ctrl 在 `traceBackResub` 的 device assert 當掉（兩節點互相 0-resub）。
- 做法：`CULS_RELAX_DIV` 跳過 `checkMffcConflict`，全部候選走 CPU greedy，每次提交前把「除數 → root」邊加入動態拓撲順序；只往 fanin 方向搜（碰到 root 即回路），把搜到的集合用小數 order 移到 root 前，間隔 < 1e-6 才 fallback 到完整 Pearce–Kelly。完整雙向版在 hyp 要 0.33 s（往前走 600 萬節點），只往回搜降到 0.01 s。
- 結果（單次 `rs -n 2`，原版／放寬／ABC）：div 45,226／45,140／45,264；hyp 212,475／207,225／207,940；mem_ctrl 46,523／46,503／46,369（4 個候選因回路被拒）；adder、multiplier 不變。hyp alg 0.07 → 0.09 s。連跑 3 次時 multiplier、div、hyp 都贏 ABC。5 個輸出通過 `cec -n`、3 次輸出逐位元組相同。報告 `outputs/qor-gap.md`，patch `work/qor-gap/relax-div.patch`。
- 建置：cthulhu 已換成 CUDA 13.3，見 [[cthulhu-cuda-13-toolchain]]。

## 防回路、輸出固定與 GPU 利用率（2026-10-25）

- `CULS_RELAX_DIV=1/2/3`：全部 CPU 逐一／混合／樂觀平行。QoR 相同；回路只出現在逐一提交部分（mem_ctrl 4、mem_ctrl_5xd 128），平行提交從未需要撤銷。混合沒用（hyp 只有 15 % 候選沒有向上除數）。
- `DynTopo` 原本為每個節點配兩個 `std::vector` 並建全圖 fanout CSR，hyp_4xd 建置 0.066 s；改成稀疏 slot 索引、CSR 延後到 fallback 才建，greedy 從 0.09 s 降到 0.04–0.05 s，輸出不變。GPU 回路偵測最多只能省 0.018 s，且 24,802 層要逐層 launch，不值得做。
- mem_ctrl 輸出不固定來自 strash `insertLevelNodes` 同層重複 key 搶雜湊槽（`insert_single_no_update`）；改 `insert_single_update_min_value` 一行修正，原版 CULS 也有此問題。binary `bin/gpuls-hyb3`。
- hyp_4xd GPU Metrics（模式 3）：整段 SM Active 27 %；strash 每層 kernel 期間 SM Active 2.8 %（平均約 237 threads，占 hyp_4xd kernel 時間約一半）；resub kernel 期間 SM Active 94 % 但 SM Issue 25 %；約 44 % 樣本沒有 kernel。
- 放大電路的 `&cec`：hyp_4xd 339 s、mem_ctrl_5xd 6 s 等價；div_5xd 超過 1 小時沒有結果（改跑 `cec -n`）。
