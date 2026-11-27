---
title: ieda1/ieda2 登入保護：SSSD worker 掛掉時 systemd 仍顯示正常，需限制使用者記憶體並做 watchdog
scope: machines/gups
machine: gups
status: active
created: 2026-11-28
updated: 2026-11-28
tags: [gups, ieda1, ieda2, sssd, freeipa, earlyoom, journald, cgroup, oom]
---

# ieda1 / ieda2：保護 FreeIPA 登入不被 EDA 作業拖死

2026-11-26 凌晨，ieda1（Rocky 8、cgroup v1、約 250 GiB RAM + 8 GiB swap）被數個同時執行、各佔 100 GiB 以上的 Innovus APR 作業把記憶體與 swap 吃光。SSSD 的 worker 被 watchdog 殺掉，主行程 PID 卻還在，所以 `systemctl` 仍認為 `sssd` 正常；`systemd-journald` 同時撞上 StartLimit 後維持 failed，日誌停更。結果 SSH 對所有 FreeIPA 學生帳號回 Access denied，再跳出密碼提示（看起來像帳號或本機存的密碼壞了，其實帳號與 IPA 都正常）。2026-11-28 已在 ieda1 恢復服務，並把同一套登入保護套到 ieda1 與 ieda2。

## 容易誤判的點

- `sssd` 主行程活著不代表能查帳號。要看 worker（`sssd_nss`、`sssd_pam`、`sssd_be`）和 `sssctl domain-status`，不要只看 `systemctl is-active sssd`。
- SSH 先 Access denied 再要密碼，常見是 NSS/PAM 查不到 IPA 使用者，不是該學生密碼過期或終端機（例如 MobaXterm）存錯密碼。先在伺服器跑 `id <student-id>` 與 IPA 帳號狀態。
- Rocky 8 是 cgroup v1。`MemoryHigh` / `MemoryMin` 不會生效；真正卡住上限的是 `MemoryLimit=`（寫入 `memory.limit_in_bytes`）。drop-in 要同時寫 `MemoryLimit=` 與 `MemoryMax=`，後者是給將來 cgroup v2 用的。
- `user-.slice` 的上限會管到 root 和所有使用者作業，所以管理員自己跑的大型 EDA 作業同樣會觸頂，這是刻意要保護登入服務。
- 這次日誌中斷是 `journald` 自己 StartLimit 失敗，和 [[recurring-hang]] 那種沒有 OOM、日誌突然沒了的沉默卡死不同。

## 2026-11-28 已套用的保護（ieda1、ieda2 相同）

- `/etc/systemd/system/user-.slice.d/memory.conf`：`MemoryLimit=230G`、`MemoryMax=230G`、`MemoryHigh=220G`。硬上限約留 20.8 GiB 給系統、FreeIPA、SSH 與 SSSD。
- `sssd` drop-in：`OOMScoreAdjust=-900`、`Restart=always`、`StartLimitIntervalSec=0`。
- `systemd-journald` drop-in：`OOMScoreAdjust=-800`、`Restart=always`、`StartLimitIntervalSec=0`。
- `/usr/local/sbin/sssd-watchdog` + `sssd-watchdog.timer`：每 2 分鐘檢查 `sssctl` 與 worker，主行程還在但子行程掛了就自動重啟 `sssd`。
- `earlyoom`：`-m 5 -s 5`；`--prefer` Innovus / Genus / `xrun` / Calibre 一類 EDA 作業；`--avoid` `sshd`、`sssd`、`sssd_*`、`systemd-journald`、`ns-slapd`、`krb5kdc`。
- `/etc/sysctl.d/99-login-protect.conf`：`vm.min_free_kbytes=1048576`（1 GiB），讓系統在記憶體見底時仍能 fork 出保護行程。

## 驗證

- `systemctl is-active sssd systemd-journald sssd-watchdog.timer earlyoom` 皆為 active。
- `id` 能解析已知的 IPA 學生帳號；`sssctl domain-status` 正常。
- `user-.slice` 的 `memory.limit_in_bytes` 已是 230G。
- 套用當下正在跑的 Innovus 作業沒有被重啟服務打斷。

相關：[[freeipa-ieda-servers]]、[[recurring-hang]]、[[eda-environment]]
