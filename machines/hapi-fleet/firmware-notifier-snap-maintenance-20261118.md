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
