---
title: CULS GPU resub profiling 與開源版基準
scope: projects/ntu-lsv-course
status: verified
updated: 2026-10-24
tags: [CULS, gpuls, resub, nsight, cuda]
---

# 專案位置

本機專案在 `<documents>/CULS-profiling`。入口為 `outputs/PROJECT.md`；`outputs/OSS-baseline.md` 為開源版對照。

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
- 三個逐層 kernel 在兩版中都跑了 4,405 次，等於電路深度。差別在開源版每層多了 thrust `copy_if`／`remove_if`，kernel 總數 30,916 次、共 68.19 ms；作者版為 13,302 次、41.21 ms。作者版看起來已把壓縮併入 `markReadyNodes`。
- conflict resolution 和 cut 計算的 kernel 在兩版中不同。
- 結論：開源版可以當改進研究的 baseline，但只能跟未修改的開源版比較，不能拿作者 binary 的數字當基準。
