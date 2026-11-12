---
title: GPU 卡死（Xid 119/154）時 process 連 root SIGKILL 都殺不掉，GeForce 只能重開機
scope: machines/hapi-fleet
tags: [gpu, nvidia, xid, sudo, reboot, valkyrie]
status: active
created: 2026-11-12
updated: 2026-11-13
---

2026-11-11 valkyrie（RTX 4090）上一個 CULS 實驗程式（swear01 的 `g7`）觸發 `Xid 119`（GSP RPC timeout）接著 `Xid 154`（GPU Reset Required）。之後那個 process 卡在 `nvidia_uvm` 的 `uvm_tracker_wait` 自旋，CPU 100%、狀態 `R`，campaign 的 `gpu.lock` 被它佔住超過 30 小時。

辨識：

- `nvidia-smi --query-gpu=pstate --format=csv` 回 `[GPU requires reset]`。
- `/proc/<pid>/status` 的 `SigPnd` 顯示 SIGKILL 已送達但一直待處理；`sudo cat /proc/<pid>/stack` 停在 `uvm_*`。
- `journalctl -k | grep -E 'Xid'` 有 119/154。注意：swear01 不在 `adm`/`systemd-journal` 群組，`journalctl -k` 只會顯示自己的訊息（有 "not seeing messages from other users and the system" 提示），`dmesg` 也被 `dmesg_restrict=1` 擋住，所以用一般權限查到「0 Xid」不代表真的沒有。要看 Xid 用 `ssh su_<host>` 跑 `journalctl -k`；一般權限下只能用 `nvidia-smi` 的 pstate（`[GPU requires reset]`）和 job 是否照常結束來判斷。

處理：

- 需要 root 時用 `ssh su_<host>`（見 [[machines/hapi-fleet/system-install-via-admin]]），不要因為 swear01 沒有 sudo 就停下來等使用者。
- root `kill -KILL` 一樣殺不掉；`sudo nvidia-smi --gpu-reset -i 0` 在 GeForce 上回 `Not Supported`。唯一解法是重開機。
- 重開前先看 `who` 和 `ps -eo user | sort | uniq -c`，有其他使用者的 process 就先問使用者。
- 2026-11-12 15:57 在 valkyrie 下 `sudo systemctl reboot` 後，10 分鐘以上 ping 和 SSH 都沒有回應（同網段其他主機正常），多半是關機時卡在這個殺不掉的 process。fleet 沒有 IPMI/BMC 這類遠端電源控制，只能到現場按電源強制重開。下次可以先試 `sudo systemctl reboot --force --force`，跳過 unit 的停止流程。這次是現場人工斷電後 17:16 開機，GPU 回到 `P8`、本次開機 0 個 Xid、`gpu.lock` 沒有持有者、hapi runner 自動起來。
- 使用者規則（2026-11-12）：自己的 job 卡住太久就主動處理，不要一直排隊等 lock。
