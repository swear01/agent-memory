---
title: CULS GPU resub profiling 與開源版基準
scope: projects/ntu-lsv-course
status: verified
updated: 2026-10-29
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

## 固定點 strash 與 lane 利用率（2026-10-25）

- strash 改成固定點迭代（所有節點同時以目前 fanin 代表元雜湊、保留最小暫定 id，重複到不變）：8 個電路只要 1–2 輪，輸出與逐層版逐位元組相同；hyp_4xd strash kernel 162 ms／49,608 launch → 9 ms／4 launch，整體 0.53 → 0.37 s。超過 32 輪退回逐層。開關 `CULS_STRASH_FIX=32`，binary `bin/gpuls-strash`。
- 單一 block 跑連續小層（`__syncthreads()` 分層）只把 strash 從 187 ms 降到 123 ms：每層仍是幾次串行的隨機 DRAM 存取，launch 不是主要成本。
- CPU conflict resolution 只搬有 gain 的 `decGraph` 列（GPU gather／scatter），hyp_4xd 0.11 → 0.07 s。
- resubDivs2 每條指令只有 2.6／32 lane 活躍；先壓縮要做的 root 再 launch 反而更慢（div_5xd 21.8 → 56.5 ms），因為配對迴圈長度有平方長尾，壓縮後每個 warp 要等 32 個裡最慢的、warp 數也少十倍。要改善需 warp 合作搜尋配對並以 ballot 取字典序第一個。
- 改完後 hyp_4xd 有 54 % 取樣沒有 kernel，瓶頸轉到 host 端（CPU greedy、`cudaFree` 約 28 ms、host 算 level 順序）。

## memory pool 與 host 空檔（2026-10-25）

- `cudaFree` 會等 device 同步再把記憶體還給驅動，hyp_4xd 閒置 149 ms 中 `cudaFree`＋`cudaMalloc` 占 66 ms。`-include mempool.h` 把兩者換成預設 stream 上的 `cudaMallocAsync`／`cudaFreeAsync`（release threshold 設最大），並讓 `cudaMemGetInfo` 把 pool 閒置部分算成可用，因為 resub 依剩餘記憶體分批，否則分批會變。開關 `CULS_MEMPOOL`，binary `bin/gpuls-pool`；8 個電路輸出不變。
- 陷阱：CULS 的開關只檢查 `getenv` 是否非空指標，`CULS_MEMPOOL=` 空字串也算開啟，曾因此把開啟結果誤當成關閉的基準。
- 只搬候選列之後，原版 resub 也是 `CULS_CONFLICT_CPU` 比 GPU 單 thread greedy 快。hyp_4xd 整體 0.37 → 0.29 s（開始前 0.53 s），剩約 0.18 s 是 resub kernel；啟動程序另有約 0.11 s 固定成本。
- cthulhu 上多個 session 共用 GPU 時計時會互相干擾（hyp_4xd 曾量到 0.81 s）；計時腳本在每次執行前後檢查其他 gpuls 程序，有重疊就重跑。`perf_event_paranoid=4`，perf 也要經 su_cthulhu `sudo -n` 單次執行。

## rs 小 kernel 的 warp 化（2026-10-28）

- getReconvCutResub／resubDivs0／resubDivs1 改一個 warp 一個 root、輸出逐位元組不變的關鍵：把原本「掃到第一個符合」的順序用 ballot 取最低 lane 重現；「最後一個勝出」（d0 的 equal 除數）則取最高 lane；列表壓縮用有序 ballot＋popc。cost 999 的槽永遠不選。
- 純 lane 串列掃描整列（128 word）會被少數需要很多 word 才能判定的列拖慢（ncu 約 14/32 lane 活躍、55% 多餘 sector）；解法是 lane 先各掃前 8 word，未決定的列由整個 warp 接手（lane = word，`__reduce_or_sync`）。同一個 pass 併入 d0 後 d01 hyp 56→21 ms；沒有混合掃描時併入反而讓 mem／div 變慢。
- 消融（拿掉 pair 搜尋、拿掉補數）是找瓶頸最快的方法：兩者都可忽略，才確定是分類掃描。
- 計時輸出 `{time} prev cmd: ... full X s; total: ... full Y s` 同一行有兩個 `full`，用 `full ([\d.]+) s` 會重複計；要錨定 `prev cmd: alg [\d.]+ s, full`。
- 以 `ssh host 'pkill -f pattern'` 會殺到自己的 ssh（pattern 出現在 bash -c 的指令列），改用 ps+grep 並排除 `bash -c`，或乾脆不殺。
- 報告 `outputs/rs-kernels2.md`，CULS-fast `5b1e1ab`。

## 10xd 在 12 GB 卡上跑完整 resyn2rs（2026-10-28）

- device malloc heap（`cudaLimitMallocHeapSize`）在第一個用 device `malloc` 的 kernel 才提交，之後 `cudaDeviceSetLimit` 失敗，所以不能在進程啟動時定大小：`b` 在 div_10xd／hyp_8xd 最多容忍 256／512 MB、mem_ctrl_10xd 要 ≥1024 MB，rf 要 2048 MB（1024 時 arbiter／log2／sin 10xd 失敗）。`thrust::sort(thrust::seq)` 在 kernel 內會用 device heap，改成就地 introsort（`f3d7abf`）。
- 每執行緒 call stack（45056 B × SM × 1536 ≈ 4.1 GB）在 mempool 之外；ISOP 後把 stack limit 降到 `evalFactoredForm` localSize＋2 KB 可釋放 2.5–3.5 GB。pool 碎裂用 `cudaMemPoolTrimTo(pool,0)`＋`culsMalloc` 遇 OOM 重試處理。平行 `b`（`fPar`）曾配置約 6 GB 用不到的 hash table，是 rf 之後 `b` OOM 的原因（`da6fc64`）。
- 驗證方式：64 格＋bench-big md5 逐位元組、5 次中位數、10xd 前綴流程逐條記累計時間；`ls` 錯機器（`/var/tmp/culs-chunk` 在 mazu）、外層 `flock` 包 pf.sh（它內部已 flock）會死鎖。
- 報告 `outputs/heap-b.md`、`outputs/rf-mem.md`。

## rs MFFC=1 root 預篩（2026-10-28）

- hyp_4xd 約 73% 的 rs root MFFC=1，只能被常數或 D0 換掉（divs1/12/2/3 的 MFFC 門檻 >1/>2/>2/>3），實際被換的僅約 1%；`simulateResubLane` 先只模擬前 G 個 word（word 彼此獨立，部分不符必推全不符，無 false negative），無候選就設 `simSize=0` 丟掉。hyp_4xd／div_5xd resyn2rs、compress2rs 約 −4.5%，mem_ctrl_5xd 與 resyn3 持平；64 格 md5 對 `dd970d4` 逐位元組相同（`cdd7877`，報告 `outputs/rs-mffc1-screen.md`）。
- 想跳過 MFFC=1 root 的 cut／collectDivisor 不行：D0 取第一個相符的 divisor，依賴完整 divisor 集合與排序。
- 重基後要重跑 64 格（base 與 new 都要在新 main 上重建）；cthulhu 不能直接 ssh 到 mazu，檔案要經本機中轉；`ssh host 'nohup ... &'` 沒重導 stdout 會讓 ssh 卡住。

## 12 GB 容量失敗與 bug 修正（2026-10-28，CULS-fast `fc5d7ee`）

- rw 分塊 cut pool 卡住（存活 cut set 多於裝置池）時，不要只靠縮小每塊大小：卡住只取決於邊界上的存活數。可用 `last[]` 差分＋前綴和算出峰值存活數，再用 `cudaMallocManaged`（溢位段 advise 成 host-preferred＋accessedBy device）的連續緩衝整趟重做；配置前要 `cudaMemPoolTrimTo(pool,0)`，因為 managed 頁不能用 stream-ordered pool 留著的記憶體。溢位的 rw 原本比平常慢 2–3 倍（後來大多是 `rwAssign` 的未合併清零造成，見下一節）。
- rf 一開頭就設 45 KB 的 stack limit，等於立刻保留約 4.1 GB；應該延到 ISOP 之前，而且要在依剩餘記憶體決定 `vSubgTable` 大小之前設（這樣大小不變，輸出逐位元組相同）。driver 會在 launch 時自動加大其他 kernel 需要的 stack。
- rs 裡一個 root 只把 `simSize` 設成 0 不夠，`resubDivs12`／`resubDivs2` 不看 `simSize`，只看 MFFC 門檻，會讀到上一批的 queue；要丟掉一個 root，得把 `vMFFCNumSaved[root]` 也設成 0。
- `pkill -f` 又一次殺掉自己的 ssh：要停背景腳本請記下 PID 再 `kill`。
- 報告 `outputs/capacity-fix.md`。

## rw 溢位路徑加速（2026-10-29，CULS-fast `2cc9e0e`）

- 慢的不是 PCIe 讀取，而是 `rwAssign`：一個 thread 負責清掉一整個 224 B 的 cut set，寫入無法合併。這在一般 rw 每次就要 0.43 s；在 managed 溢位記憶體上更會變成每塊最多約 1 s 的 first-touch fault，每次溢位 rw 共 2.6–3.9 s。改成一個 thread 寫一個 `int4`（每個 set 14 個 thread）之後，降到 0.03 s（一般）／0.09–0.10 s（溢位）。hyp_8xd resyn2 從 39.9 s 降到約 26 s，輸出逐位元組相同。
- 對 managed 緩衝做 `cudaMemPrefetchAsync` 沒有好處，而且有副作用：預取裝置段之後，下一個指令可用的顯存少了 40–100 MB，結果讓後面原本放得下的 rw 也變成溢位（hyp_8xd abcs resyn2rs 的溢位次數從 1 次變 3 次）。只預取 host 段無害，但也沒有加速；最後完全不做 prefetch。教訓：先把存取改成合併寫入，再判斷是否需要 prefetch；改用 managed 記憶體的 API 之後，要看下一個指令的 free memory 有沒有變。
- 剩下的溢位成本（每次比一般 rw 多約 0.7 s）分成三塊：flow kernel 經 PCIe 讀 host 端 cut set 約 +0.3 s；evaluate 約 +0.1 s；其餘是分塊數變成 50–135 塊所增加的同步與 thrust 呼叫。這些屬於溢位本身的成本。
- 報告 `outputs/rw-overflow-opt.md`（CULS-profiling `9b17ccf`）。

## host 空檔第一輪、容量判斷、裝置相依輸出（2026-10-29，CULS-fast `28b7d63`）

- host 空檔要先量再改。nsys 的 CPU 取樣能把空檔歸到函式，`CULS_PHASE=1` 則印每段 ms。這輪的大頭都是「每次呼叫新配置 graph 大小的 host vector」：
  - first-touch page fault 加上 zero fill；
  - static vector 變大那次 0.4 s 的尖峰。
- 改用 pinned 池（`CulsHostBuf`）、把排序搬到 GPU（cub radix，key 為 (level, id)），三個 10xd 電路快了 6–16%，輸出逐位元組相同。
- 「整個陣列放得下」的判斷必須算進之後還會配置的暫存。multiplier_10xd 的 cut 陣列 5.4 GB 放得下，只剩 0.44 GB，接著 sort 暫存就配置失敗了。修法是保留量與 n 成正比（40 B／節點＋64 MB）。
- 不要用固定的 1 GB 保留量：模擬小卡時（`CULS_FAKE_TOTMB=3072`），連 adder 也被推進 chunk 路徑，然後結束。
- rw 輸出會因 GPU 不同而變：`tdBlocks = 2 × SM 數` 決定 top-down／bottom-up 的分界。只要演算法分界取決於 SM 數、occupancy 或 L2，換卡就會換輸出（同一張卡仍然是決定性的，所以 5 次重跑抓不到）。
  - 修法：分界改成常數。另加 `CULS_FAKE_NSM`／`CULS_FAKE_TOTMB` 測試 hook，在單卡上用 `md5cfg.py` 跑 64 格掃描。
  - mazu 的 RTX 4060 只有 24 SM，修正前它的輸出本來就和 4070 Ti 不同。跨機比 md5 前，要先確認 SM 數是否相同。
- 小坑：
  - `git cherry-pick` 沒有 `-q`。
  - 從 Mac `scp` 過去的腳本沒有 +x，`nohup ./x.sh` 會 Permission denied，用 `bash x.sh` 執行。
  - `ssh host 'nohup … &'` 仍會卡住 ssh；改用 `(setsid nohup … < /dev/null &)`。
- 報告 `outputs/host-gap-round1.md`（CULS-profiling `d7b3d3e`）；issue #3 已關，#6、#7 開著。

## strash GPU 分層、device heap 耗盡（2026-10-29，CULS-fast `5ae3f0b`）

- 比較「搬到 GPU 划不划算」時，要把搬回和上傳也算進去：
  - strash 的 host 分層單看只有 0.077 s，比 GPU spin（0.1 s）快；但加上拷回和上傳後就輸了。
  - 改成 `structLevelSpin` 後，每次 strash 從 70–260 ms 降到 17–20 ms；連 24,801 層的 hyp_8xd 也是 GPU 勝，因為上一次 strash 已依 level 排好 id。
  - 只有剛讀入的 AIG（id 未依 level 排）第一次較慢。
- kernel 內的 device `malloc` 在 Release（`-DNDEBUG`）下失敗會回傳 NULL，`assert` 被拿掉，結果是寫到 NULL 加位移的位址。同時駐留的 warp 越多（A100 108 SM），heap 越早用完：
  - 會表現成「只在某張卡上必定 illegal access」，在小卡上則是偶發。
  - 修法：失敗的 warp 記下工作，host 分批重做。
  - 重現：把 `CULS_HEAP_MB` 調小。
  - 別先假設是 SM 數，先用 fake NSM 排除。
- stdout 寫到檔案時是整塊緩衝，卡住的 process 的 log 只會停在最後一次 stderr（這裡只有 rw 寫 stderr）。查卡住要用 `stdbuf -oL -eL`。`cuda-gdb -p` attach 到不是在 cuda-gdb 底下啟動的 process 只會回報「No CUDA kernels」，看不到 device 狀態；要 warp 的 backtrace 得從一開始就在 cuda-gdb 下執行。
- `&cec -y` 對 sqrt_10xd resyn2 回報 UNDECIDED（428 s），`&cec -x` 459 s 證明等價。CEC 要讓多種引擎競賽，見 `cec-fast.sh`。
- 報告 `outputs/host-gap-round2.md`（CULS-profiling `751340b`）；issue #8（rf heap）、#9（A100 偶發卡住）。

## A100 偶發卡住：truth table 不再用 device malloc（2026-10-29，CULS-fast `0fa312b`）

- 用 stderr 的 rw 行（`0 in EvaluateNode`）對照 flow 步驟，就能定位卡住點：先在別台卡用 line-buffered 跑一次完整 log，看第 N 個 rw 之後是哪一步。log2_10xd resyn2 的 `Max Level: 353` 是第 3 個 `b`。
- A100 的 mem_ctrl_10xd resyn2 卡在 rf 的 cone 數和 truth table 計時之間，GPU 100%，host 停在 libcuda。那段唯一不一般的是每個 warp 的 device `malloc`，而 heap 在 108 SM 上會用完。注意 `CULS_MEMOPT` 下 heap 預設 128 MB，但 `main()` 會在 `-c` 腳本含 rf（resyn2、resyn2rs…）時改成 2048 MB；先看 `main.cpp` 再引用 heap 大小。模仿它的 malloc 壓力測試在 4070 Ti／4060 上跑 100 次都沒卡，沒能在本地重現。
- 修法：不追究 malloc，直接換成預先配置的 arena（`CULS_RF_ARENA_MB`，預設 min(free/4, 1 GB)），用 `atomicAdd` 切給 warp，分段 launch、每段歸零，下一段大小依上一段實際用量推算，放不下的 warp 下一輪重做。輸出相同，truth 時間 0.22→0.18 s。
- `pgrep -f "bench3.py gpu"` 寫在 `ssh host '...'` 的命令列裡會比對到自己的 `bash -c`，永遠回報「有在跑」；和 pkill 一樣，要放進腳本檔再執行。
- 報告 `outputs/a100-hang-round3.md`（CULS-profiling `6ef9850`）；關閉 #6、#8，#9 等 A100 重跑確認。
- build 的 `-arch` 改由 CMake cache 變數 `GPULS_CUDA_ARCH` 決定（預設 `native`，CULS-fast `8b55c0b`）；在 A100 上 build 不必再改 CMakeLists。

## 同一 warp 內的 spin-wait 與 BSYNC（2026-10-29，CULS-fast `239c496`、`10fc919`）

- 一條 lane 等同 warp 另一條 lane 發布的值，在 SASS 裡常是：等待迴圈包在 `BSSY B1`／`B0` 裡，被等的 lane 要先過 `BSYNC` 才能發布；能不能過全看排程器。4070 Ti 上從沒卡過，但這不是保證，可能就是 A100 偶發卡住（#9）的原因。用 `cuobjdump -sass` 看等待迴圈和發布的 `STS`／`STG` 之間有沒有 `BSYNC`。
- 修法一（`structLevelSpin`）：整個 warp 同步輪詢，每輪 `__syncwarp()` + `__all_sync()`，不提早 return。代價小（hyp_8xd 一次 resyn2 的 strash level 169→193 ms）。
- 修法二（balance `parPhase2`）：讓 warp 一次領同一層的 32 張票（每層補到 32 的倍數），依賴必在更深層、更早的批次，所以不會等自己 warp 的 lane。改成每 warp 只留一條 lane 雖然也安全，但 phase 2 慢 1.6–1.7 倍。
- balance 的 `CULS_BAL_PAR` 預設是開的（`culsOpt(..., 1)`），baseline commit 訊息寫「PAR default 0」已過時。
