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

# 症狀與根因

- 同學回報工作站「用一用就當掉、連進來要等很久」。Cthulhu 上每個使用者的 `snap.firmware-updater.firmware-notifier.service`（user service，由同名 timer 觸發）都只活 0–2 秒。
- 根因：notifier 一定要呼叫 `org.freedesktop.Notifications`；SSH／VS Code 等無桌面 session 沒有通知服務，拋 `DBus.Error.ServiceUnknown` 後 exit 255，systemd 不停重啟。
- 更新到 rev 262 **不能**修好：2026-11-18 在 Cthulhu 實測 262 一樣 crash（NRestarts 11）。utux 上 262 長期正常，只是因為該處跑在有桌面的 session。

# 修法（已套用）

- `sudo systemctl --global disable snap.firmware-updater.firmware-notifier.timer`，再對已登入使用者 `systemctl --user stop` timer 與 service。
- 已套用：Cthulhu、Valkyrie、Athena。Mazu、Zeus 沒裝 firmware-updater。utux 是單一使用者機器，改用 `systemctl --user mask` 在 utux 帳號關掉（不需要 sudo）；2026-11-19 也已用 `sudo systemctl --global disable` 關掉全域 timer（utux 帳號有 NOPASSWD sudo）。
- 免密 sudo 的作法：Cthulhu 等機器靠 `/etc/sudoers.d/swear02-nopasswd`（`swear02 ALL=(ALL) NOPASSWD: ALL`），`dvlab-admin` 只給 `%admin` 需要密碼的 sudo。utux 有接 NIS（domain `ee.ntu.edu.tw`，swear02 在 admin 群組），但 `/home` 不是 NFS，swear02 沒有 home 和 key，所以無法用 key 登入。
- `snap stop --disable --users=all firmware-updater.firmware-notifier` 在多使用者機器上會因 per-user agent 逾時失敗，不要用。
- snap refresh 後要確認 `/etc/systemd/user/timers.target.wants/` 裡沒有 firmware-notifier timer。

# 2026-11-18 更新決策（使用者指示「全部更新」）

- 解除 Cthulhu、Valkyrie、Athena、Mazu 的 `snap refresh --hold`（原本 hold forever，firmware-updater 卡在 rev 226），四台 snap 全部更新到最新。這推翻了 [[machines/hapi-fleet/ubuntu-gpu-maintenance-plan]] 裡「四台 Snap refresh hold」的狀態。
- apt：Zeus、Valkyrie、Athena、Cthulhu、Mazu 全部 full-upgrade，kernel `7.0.0-43`。使用 `NEEDRESTART_MODE=l`、`--force-confold`，不重啟服務。
- NVIDIA 驅動仍維持 hold（使用者指示「先不處理」）：把原本 hold 清單 unhold，再只 hold `*580-server*`、`nvidia-*`、`libnvidia-*`、`xserver-xorg-video-nvidia*`、`nvidia-firmware*`。CUDA userland 與其他原本被 hold 的套件（如 libcurl）已升級。
- 剩下的待升級套件只有 driver hold 和 Ubuntu phasing（sssd、fwupd 等）；phasing 不要強制裝。
- APT 自動更新（`20auto-upgrades`）仍是 `0`；改成開啟的操作被權限分類器擋下，需要使用者明確同意。
- 已排程五台（Cthulhu、Zeus、Valkyrie、Mazu、Athena）於 2026-11-19 09:30 `shutdown -r` 換新 kernel。Athena 的 TZ 顯示 PST，但時鐘與台北一致，排程的 epoch 跟其他台相同。

# Snap Store 不穩

- 2026-11-18 約 22:25 起，`api.snapcraft.io` 時好時壞（connection refused／逾時）；從實驗室和外部網路測都一樣，snapcraft.io 網站與 apt 套件庫正常。
- snap refresh 是整批 change，任一步失敗就整批回滾；遇到 store 不穩時，每台每 90 秒重試，大約 3–5 次會成功。
- snap 沒有公開鏡像：連 `snap download` 也需要 API，唯一的替代方案是 Canonical Enterprise Store／Snap Store Proxy。

# 2026-11-19 後續：提前重開與 NIS 中斷

- 使用者要求閒置的機器先重開。02:12 對 zeus 和 utux 下 `systemctl reboot`（先 `shutdown -c`）。到 02:45 zeus 都沒有回來，ping 不通、22 port 關閉；fleet 沒有遠端電源控制。
- zeus 是 NIS master（192.168.1.201）。它一掛，各 client 的 SSH 認證能通過，但登入卡在 NSS：utux 77 秒、cthulhu 超過 120 秒被 sshd LoginGraceTime 斷線。重開 NIS master 之前，要先確定能到現場處理，或先排好其他台的時間。
- 七天 SSSD 快取並沒有失效：根因是 NSS 順序 `files nis sss systemd`，ypbind 還活著、不斷重新 bind，每次查詢都先等 libnss_nis 的 RPC 逾時（寫死、無法設定）。8/23 的驗證是 stop ypbind，會立刻 UNAVAIL，所以沒測出這個問題。使用者的要求：先查線上、短逾時（1–3 秒）、再用快取。建議作法是 `files sss systemd`，由 SSSD proxy 查 NIS 並處理 offline；這也符合 SpyGlass「systemd 排最後」的限制。
- NVIDIA 595（使用者授權，Zeus 除外）：athena 已裝好 `nvidia-driver-595-server`、`linux-modules-nvidia-595-server-generic`、`-7.0.0-43-generic`，還沒重開。valkyrie、mazu 因 SSH 卡住還沒裝。另一個 session 排的 `nvidia-595-swap.timer` 是 transient timer（重開就消失，而且只裝 -38 的模組）。
- 後續交給 HAPI 上的 Codex session「DVLab NIS 中斷復原」（`f2908b12-d06d-42ca-8208-c4dc92ad0faf`）：zeus 恢復、NSS／SSSD 短逾時修正、595 與重開、記憶更新。NIS 變更與 DROP 驗收見 [復原 runbook](ubuntu-upgrade-recovery-runbook.md#2026-11-19-部署與-drop-驗收)，NVIDIA 套件狀態見下節。

# 2026-11-19 NVIDIA 安裝收尾

- Cthulhu 與 Valkyrie 在 09:15 啟動預排的 595 安裝。09:30 重開時，systemd 停掉仍在執行的安裝 unit。兩台的 `nvidia-compute-utils-595-server` 都停在 half-configured，其他相關套件已解包且仍有 initramfs 等 trigger 待跑。Valkyrie 的 journal 顯示 `adduser` 09:28:32 開始建立服務使用者，09:29:46 才開始建群組。重開後兩台都缺本機 `nvidia-persistenced` 帳號。當時 Zeus 仍失聯。日誌可證明安裝被截斷，未能證明每段帳號查詢延遲的來源。
- 回線後兩台均已執行 `dpkg --configure --pending`，套件的 postinst 建回帳號並完成待處理的 trigger。再啟動 `nvidia-persistenced`。兩台的 `dpkg --audit` 都無輸出，595 套件與 `7.0.0-43` 模組均為 installed，服務 active，`nvidia-smi` 及 CUDA 裝置初始化都通過。
- Mazu 在確認沒有 CUDA 工作及本機圖形登入後，短暫停掉 GDM 登入畫面和 persistence daemon。正常卸載 580 模組後安裝 595 與正在執行的 `7.0.0-43` 對應模組，再載入 595 並恢復服務。用 boot ID 和先前的 SSH／編輯器程序確認主機與連線持續運作，HAPI 入口 HTTP 200。Mazu 的 dpkg audit、GPU 與 CUDA 初始化均通過。Athena 已在 595，Zeus 的 Pascal GPU 維持 580。
- 以套件安裝結束及 dpkg audit 當成重開前的檢查條件。先確認正在執行的核心有對應的 NVIDIA 模組，再驗證載入版本、套件狀態、服務和 CUDA；`nvidia-smi` 成功只能證明目前 GPU 與驅動通訊正常。
