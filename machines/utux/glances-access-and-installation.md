---
title: Utux 本地管理帳號與 Glances 安裝驗證
scope: machines/utux dashboard monitoring
status: deployed and API verified 2026-10-28; reboot untested
updated: 2026-11-19
---

- Mac 的 SSH alias `utux` 使用本地 `utux` 帳號，可登入；2026-11-19 實測 `sudo -n` 可用（`NOPASSWD: ALL`），不必再從 Vaultwarden 項目 `DVLab · utux 本地管理帳號` 注入密碼。utux 的 Minecraft 伺服器 `vault-hunters.service`（`/srv/minecraft/vault-hunters`）已於 2026-11-19 依使用者要求 `disable --now`，開機不再啟動。Zeus 的既有 `swear02` key 被拒，未修改 SSH key 或安全設定。
- Glances `4.5.8` 位於 `/opt/glances-venv`；`glances-web.service` 已 enabled 且 active，僅綁 LAN 位址的 `61208`。從 utux 與 zeus 讀回 load／mem／fs API 均 HTTP 200。
- 乾淨 Ubuntu 沒有 `curl`；Linux installer 的 self-test 因而印出空結果，仍 exit 0。PR #5 在既有 apt 依賴加入 `curl`，已合併為 `9f4d1f6` 並部署到 clean zeus checkout；補裝後 API 與 web-asset 自測皆 200。不能把 installer exit 0 當作服務成功。
- Live overview 與內建 browser 顯示 utux online、20 cores、32 GB；SSH enrichment／process age 尚無資料，filesystem fallback 可用。
- 未驗證整機重開機；service active／enabled 與 API 成功不能代替 reboot proof。

- 2026-11-19 套件同步：full-upgrade 到 kernel `7.0.0-43`，再補上其他台 ≥4 台有的 manual 套件（排除 grub、sssd、kernel meta），加上 cthulhu 的 Docker、LLVM apt 來源與 i386 架構；libxp6:i386 照 [[machines/hapi-fleet/eda-install-inventory]] 用官方 DEB 安裝。apt 版 glances 4.3.3 也一起裝了，`/opt/glances-venv` 的 glances-web 不受影響。
