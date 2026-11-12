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

# 仍待補（需要 sudo，Athena 沒有免密碼 sudo）

- [ARM memory compiler 依賴](../hapi-fleet/eda-install-inventory.md)：`libxt6t64:i386`、`libxtst6:i386`、`libxp6:i386`、`multiarch-support` 皆未安裝。
- [SpyGlass NSS 順序](../hapi-fleet/synopsys-spyglass-linux7-compatibility.md)：Athena 是 `passwd/group: files systemd nis sss`，需把 `systemd` 移到 NIS／SSSD 後方（cthulhu 為 `files nis sss systemd`）。
