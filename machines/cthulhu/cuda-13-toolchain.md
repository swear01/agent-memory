---
title: cthulhu CUDA 13.3 工具鏈與舊 CUDA 專案的編譯
scope: machines/cthulhu
machine: cthulhu
status: verified
updated: 2026-10-25
tags: [cuda, nvcc, thrust, cccl, gcc]
---

2026-10-25 確認：cthulhu 的 `/usr/local/cuda` 指向 `/usr/local/cuda-13.3`，原本的 CUDA 11.8 已移除；預設 GCC 是 15.2，`/usr/bin` 另有 `gcc-10` 到 `gcc-15`。之前用 11.8 編好的 binary 仍可執行。

為 CUDA 11 寫的專案（例如 CULS）要在這裡重新編譯，遇到的問題與解法：

- `nvcc fatal : Value 'compute_60' is not defined for option 'gpu-code'`：CUDA 13 不支援 sm_60，改成只編 `-arch=sm_89`（RTX 4070 Ti）。
- cmake 找不到 CUDA compiler：先 `export PATH=/usr/local/cuda/bin:$PATH`。
- GCC 15 的 libstdc++ 不再間接 include `<cstdint>`、`<cstdio>`：改用 `CC=gcc-11 CXX=g++-11 CUDAHOSTCXX=g++-11`，或在原始碼補 include。
- CCCL 3（Thrust）不再間接提供 `thrust::tuple`、`thrust::get`：補 `#include <thrust/tuple.h>`。
- `thrust::identity` 已移除：改用 `cuda::std::identity()`（`#include <cuda/std/functional>`）。
- 從別的目錄 `cp -a` 過來的 build 目錄，`CMakeCache.txt` 仍指向舊來源路徑，要刪掉重建。
