---
title: zeus 用 runfile 系統安裝 CUDA 12.9.1，需要 g++-14 和 glibc 2.43 header 修補
scope: machines/zeus
tags: [cuda, nvcc, gpu, 1080ti, runfile, glibc, gcc, sudo]
status: active
created: 2026-10-26
updated: 2026-10-26
---

zeus 是 GTX 1080 Ti（Pascal，sm_61），Ubuntu 26.04，glibc 2.43，系統 gcc 15，driver 580.178.04（`nvidia-smi` 顯示 CUDA 13.0，這只是 driver 支援的上限，不代表裝了 toolkit）。CUDA 13 移除了 sm_75 以前架構的離線編譯，所以依照 [[machines/hapi-fleet/ubuntu-gpu-maintenance-plan]]，zeus 維持 CUDA 12.9。

## 現況（2026-10-26）

- 用管理員帳號 `ssh su_zeus` 安裝（規則見 [[machines/hapi-fleet/system-install-via-admin]]）。安裝檔是 `cuda_12.9.1_575.57.08_linux.run`，有用 NVIDIA 的 md5sum.txt 驗證。安裝參數 `--silent --toolkit --toolkitpath=/usr/local/cuda-12.9 --override`，只裝 toolkit，driver 沒有變動。`/usr/local/cuda` 指向 `cuda-12.9`，runfile 自己建立了 `/etc/ld.so.conf.d/cuda-12-9.conf`。NVIDIA 的 ubuntu2604 repo 裡沒有 12.x 套件，所以只能用 runfile。解除安裝用 `/usr/local/cuda-12.9/bin/cuda-uninstaller`。
- `/etc/profile.d/cuda.sh` 把 `/usr/local/cuda/bin` 加進 PATH，內容和 mazu 相同，只有 login shell 會套用。
- `/etc/environment` 的設定：
  - `NVCC_PREPEND_FLAGS="-ccbin /usr/bin/g++-14"`：12.9 不支援 gcc 15。使用者自己再指定 `-ccbin` 時以使用者的為準，nvcc 只會多一行 incompatible redefinition 警告。
  - `NVCC_APPEND_FLAGS="-Wno-deprecated-gpu-targets"`：關掉 12.9 編譯 sm_61 時「sm_75 以前架構將被移除」的警告。使用者明確要求保留。
- glibc 2.43 把 `rsqrt`、`rsqrtf`、`sinpi`、`sinpif`、`cospi`、`cospif` 宣告成 `noexcept`，和 CUDA 12.9 `targets/x86_64-linux/include/crt/math_functions.h` 的宣告衝突，連 g++-13 和 g++-14 都會編譯失敗。已經在這 6 行加上 `noexcept (true)`，原檔備份為 `math_functions.h.orig`。**重新安裝或更新 12.9 會覆蓋這個修補，要重新套用。**
- 驗證方式：用 swear01 開 login shell，`nvcc -arch=sm_61` 編譯，static 和 `-cudart shared` 都能在 1080 Ti 上正確執行，`cuobjdump` 顯示 sm_61 cubin。
- 中途暫裝的 apt `nvidia-cuda-toolkit` 12.4，連同那次交易帶進來的 49 個套件都已 purge，列表取自 `/var/log/apt/history.log`。`libnvfatbin12`、`libnvidia-ml-dev`（hold）、`libnvjpeg12` 在那之前就裝了，沒有移除。
- swear01（`ssh zeus`）沒有 sudo。swear01 的 conda 預設 pkgs 目錄 `/srv/shared/conda/pkgs` 不可寫入；不要在 conda 另裝 CUDA。
