---
title: firmware-notifier 重啟迴圈與 2026-11-18 snap／apt 全面更新
scope: machines/hapi-fleet
project: hapi
status: active
confidence: high
created: 2026-11-19
updated: 2026-11-19
tags:
  - snap
  - firmware-updater
  - systemd
  - apt
  - reboot
---

# Snap firmware-notifier crash loop 與分批維護

- Headless SSH／VS Code session 沒有 `org.freedesktop.Notifications`，firmware-notifier 會 `DBus.Error.ServiceUnknown`、exit 255 後重啟。Rev 262 在有桌面的 utux 可用，在 headless Cthulhu 仍 crash。改善方式為 `systemctl --global disable snap.firmware-updater.firmware-notifier.timer` 並停目前登入者的 user timer/service；改 firmware 本體與停 notifier 是兩件事。
- 已處理 Cthulhu、Valkyrie、Athena 與 utux；Mazu、Zeus 當時未裝此 snap。utux 也有使用者層 mask。`snap stop --disable --users=all` 在多使用者主機可能等 user agent timeout。Snap refresh 後重新查 `/etc/systemd/user/timers.target.wants/` 和 active user units。主機 admin principal 須從 live SSH/sudo 檢查，避免把 utux 的 NIS admin 群組誤當成已配置同一把登入 key 或 NFS home。
- 2026-11-18 的 fleet 更新已解除四台 Snap hold，APT 自動更新保持 0。手動升級先備份 config、安排 needrestart 行為，依 Ubuntu phasing 保留當下未輪到的更新。Snap Store API 偶發拒絕或 timeout 時，應查外部對照、change log 與 rollback，延遲重試；`snap download` 同樣依賴 Store API。不要把單一網站仍通當成 API 已恢復。
- Zeus 是唯一 NIS master，重開前安排本機救援，逐機核對 SSH/NSS，不用舊「stop ypbind」測試代表黑洞。最新的 NSS/PAM／DROP 驗收見 [NIS 復原](ubuntu-upgrade-recovery-runbook.md#2026-11-19-部署與-drop-驗收)。

## 2026-11-19 NVIDIA 安裝收尾

- Cthulhu/Valkyrie 排程重開截斷仍跑的 driver postinst，`nvidia-compute-utils-595-server` half-configured，本機 `nvidia-persistenced` 帳號也未建好。回線後用 `dpkg --configure --pending` 完成 pending 和 triggers，再驗帳號/service、dpkg audit、執行中 kernel 的 module、loaded driver、實際 CUDA 初始化。不要僅以某個時間排程認定安裝已完成。Transient timer 重開即消失，亦可能仍指舊 kernel。
- Mazu 當時確認無 CUDA 或本機圖形工作後，短暫停 GDM/persistence，正常 unload 舊模組並裝載對應執行中 kernel 的新模組，保留 boot 和 SSH／編輯器程序後驗 HAPI 與 CUDA。僅適用已清空 workload 的維護。Ada 595 與 Zeus Pascal 580 的 modalias／Toolkit 區別見 [GPU 維護](ubuntu-gpu-maintenance-plan.md)。

[整理前的完整證據與逐批紀錄](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/machines/hapi-fleet/firmware-notifier-snap-maintenance-20261118.md)
