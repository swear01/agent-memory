---
title: Athena 送修歸還與回機後狀態
scope: machines/athena
status: active
updated: 2026-11-12
---

# 目前狀態

2026-10-17 至 2026-11-12 送修；2026-11-12 使用者確認修好歸還，已恢復一般 fleet 操作範圍，不再排除 SSH、探測、部署或同步。

# 回機核對（2026-11-12）

- SSH 正常；hostname `athena`、LAN `192.168.1.202`、Ubuntu 26.04、kernel `7.0.0-31`（其他四台為 `7.0.0-38`，未升級，需要 sudo）。`/apps` 為 automount，`/usr/local/bin/abc` 仍在。
- 送修期間本機 `/var/tmp` 被清空：`/var/tmp/hapi-runner/runner.env`、`/var/tmp/hapi-athena/settings.json` 與 codex／opencode／cursor 本機 state 目錄都消失，`hapi-runner.service` 卡在 `Failed to load environment files` 的 auto-restart 迴圈。NFS home 上的 unit、binary（`0.30.7.7`）與 `hapi-runner-sync` 都還在。
- 修復：從 cthulhu 複製 `runner.env`（只改 `HAPI_HOME=/var/tmp/hapi-athena`）、從 cthulhu 的 `settings.json` 換回原 machineId `68d64dd8-4b63-4b86-956a-ae807d75b6f1`（不這樣做 runner 會註冊成新機器）、重建並從 `.nfs-seed` 種回本機 state 目錄。重啟後 active/running、KillMode=process、MainPID 等於 `runner.state.json` pid，hub 顯示該 machineId active。
- 教訓：任何本機 `/var/tmp` 被清掉的 NAS 機器都要先重建 `runner.env` 與帶原 machineId 的 `settings.json`，再啟動 runner。

# 回機補齊（2026-11-12，用 `su_athena` + `sudo -n`）

- `apt-get full-upgrade`（345 升級、0 移除）→ Ubuntu 26.04.1、kernel `7.0.0-43-generic`（比其他四台的 `-38` 新）、systemd `259.9`（其他為 `259.5`）。NVIDIA `580.178.04` DKMS 已為 `-43` 編好，RTX 4090 正常。升級中 openssh 重啟，SSH 會拒絕連線數分鐘，屬正常。重開機後 dpkg audit 乾淨、無 failed unit。
- [SpyGlass NSS](../hapi-fleet/synopsys-spyglass-linux7-compatibility.md) 改成 `files nis sss systemd`，備份 `/var/backups/spyglass-nss-20261112-155522/`。
- [ARM memory compiler i386 依賴](../hapi-fleet/eda-install-inventory.md) 已裝；另補 earlyoom（同 mazu 設定與 `DynamicUser=no` drop-in）、glances、python3-bottle、finger、screen、texlive-latex-extra，與 mazu／cthulhu／valkyrie 一致。
- 新坑：`systemd-oomd` 拖慢登入約 2 分鐘；已在 Athena、mazu、cthulhu、valkyrie 移除套件並 mask，見 [earlyoom 筆記](../hapi-fleet/earlyoom-nis-dynamicuser.md)。
- 仍未做：swear01 的 `claude-relink` crontab（與 mazu 相同的一行）安裝時被權限檢查擋下，待使用者決定。
