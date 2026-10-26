---
title: zeus 的 nvcc 用 apt 系統安裝 CUDA 12.4，1080 Ti 不能用 CUDA 13
scope: machines/zeus
tags: [cuda, nvcc, gpu, 1080ti, apt, sudo]
status: active
created: 2026-10-26
updated: 2026-10-26
---

zeus 是 GTX 1080 Ti（Pascal，sm_61），Ubuntu 26.04，driver 580.178.04（`nvidia-smi` 顯示 CUDA 13.0）。原本只有 driver，沒有任何 CUDA toolkit，所以沒有 `nvcc`。

- CUDA 13 移除了 sm_75 以前架構的離線編譯，1080 Ti 要用 12.x 的 nvcc。driver 580 可以跑 12.x 編出來的程式。
- 2026-10-26 用管理員帳號（`ssh su_zeus` → swear02，sudo 不需要密碼）執行 `apt-get install nvidia-cuda-toolkit`，裝的是 Ubuntu 套件 12.4.131，`/usr/bin/nvcc`。用 swear01 執行 `nvcc -arch=sm_61` 編譯並跑測試 kernel，結果正確。
- 使用者要求這類工具統一用管理員帳號系統安裝，不要在 swear01 的 conda 裡另外裝一份，避免檔案重複。swear01（`ssh zeus`）沒有 sudo。另外，swear01 的 conda 預設 pkgs 目錄 `/srv/shared/conda/pkgs` 不可寫入。
- 「sm_75 以前架構將被移除」的 deprecation 警告只有 CUDA 12.9 會出現，12.4 編譯 sm_61 不會出現（已用 `env -u NVCC_APPEND_FLAGS` 驗證）。2026-10-26 依使用者要求在 `/etc/environment` 加了 `NVCC_APPEND_FLAGS="-Wno-deprecated-gpu-targets"`，nvcc 會直接讀這個變數，新的 SSH session 也會套用。這個設定目前用不到，是給以後升級版本時用的。使用者已確認要保留，不要因為目前用不到就刪掉。
- 如果需要 CUDA 12.9：NVIDIA 沒有 ubuntu2604 的 apt repo；系統 gcc 15 超過 CUDA 12.x 支援的版本，必須用 `-ccbin g++-13`（系統有安裝 gcc-6～15）。
