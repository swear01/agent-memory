---
title: 實驗室主機的系統工具統一用管理員帳號安裝，不在 swear01 另裝一份
scope: machines/hapi-fleet
tags: [install, sudo, admin, cuda, conda, fleet]
status: active
created: 2026-10-26
updated: 2026-10-26
---

使用者規則（2026-10-26）：在實驗室主機上，給大家共用的工具（toolkit、compiler、CLI、系統套件，例如 CUDA/nvcc）一律用管理員帳號在系統層級裝一份。不要在 swear01 的 conda、`~/.local` 或 home 目錄裡另外裝一份，這樣才不會產生重複的檔案。

- 管理員帳號是 `ssh su_<host>`（User `swear02`）。2026-10-26 確認過 `su_zeus`、`su_valkyrie`、`su_cthulhu`、`su_mazu` 都可以 `sudo -n`。`su_athena` 沒有測，因為送修中，見 [[machines/athena/repair-status]]。
- 一般帳號 `ssh <host>`（swear01）在這四台都沒有 sudo。swear01 沒有 sudo 不是在使用者層安裝的理由，應改用 `su_<host>`。
- 安裝前先看 [[machines/hapi-fleet/ubuntu-gpu-maintenance-plan]] 這類既有的維護計畫，裡面可能已經定好版本和安裝來源；NVIDIA、CUDA、container toolkit 有 apt-mark hold。
- 單一專案自己的相依套件（專案的 venv、conda env）不在這條規則的範圍內。
- 實例：[[machines/zeus/cuda-toolkit-system-install]]。
