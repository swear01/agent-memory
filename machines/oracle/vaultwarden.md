---
title: Oracle VPS 上的 Vaultwarden 與 agent 取密碼的 vault wrapper
scope: machines/oracle
tags: [vaultwarden, tailscale, caddy, bitwarden-cli, oci]
status: active
updated: 2026-10-21
---

本機 SSH alias `oracle`（Ubuntu 24.04 arm64，tailnet hostname `swever`）跑 Vaultwarden，只能從 tailnet 用 `https://swever.<tailnet>.ts.net` 存取。設定檔在本機 `<home>/Documents/vault-infra`（server/、client/vault、README 施工步驟）。

- Vaultwarden：Docker compose 在 `/opt/vaultwarden`，`network_mode: host` + `ROCKET_ADDRESS=127.0.0.1` `ROCKET_PORT=8000`，不發佈 port，避免 Docker NAT 規則混進 OCI 的 iptables。`SIGNUPS_ALLOWED` 預設 false，只有建帳號時用 `sudo SIGNUPS_ALLOWED=true docker compose up -d` 打開。
- Caddy（apt 2.6）在 host，憑證由 tailscaled 提供，需要 `/etc/default/tailscaled` 的 `TS_PERMIT_CERT_UID=caddy`，且 tailnet admin 要開 MagicDNS 與 HTTPS Certificates。第一次 TLS handshake 會現場取憑證，可能超過 10 秒，curl 測試要給長 timeout。
- 防火牆：OCI Ubuntu 映像自帶 `netfilter-persistent` 的 iptables（INPUT 最後 REJECT，只開 22/43005/53346）。不要裝 ufw；Tailscale 自己加 ts-input 放行 tailscale0。公網 80/443 實測不通。
- 用 `pkill -f "tailscale up"` 會連帶殺掉自己那條含相同字串的 ssh 命令；改用 `pkill -f "^tailscale [u]p"`。`tailscale up` 的授權 URL 過期時 tailscaled log 會出現 `http 410: auth path not found`，瀏覽器卻仍顯示 Login successful，要重新產生 URL。
- Tailnet policy 已改成「我的裝置可連全部，其他成員只連 autogroup:self」，並有 tests 確認其他成員連不到 swever:443/22。Tailscale API OAuth client secret 存在 macOS Keychain service `tailscale-api`。
- Agent 取密碼：`vault run VAR=item[#field] -- cmd`（不印出）或 `vault get item [field]`。使用者刻意選擇單一帳號、agent 可讀全部密碼。從瀏覽器匯入的項目常同名（同網站多帳號）且名稱含 `:`，所以 field 分隔用 `#`，同名時錯誤訊息列出 id 與 username，改用 id 查。
- 備份：restic → B2 bucket `SwearPassword`（S3 endpoint `s3.us-west-004.backblazeb2.com`，路徑 `vaultwarden`），`/etc/cron.d/vaultwarden-backup` 每天 03:17 UTC 跑 `/usr/local/sbin/vaultwarden-backup`。B2 金鑰在 vault 項目 `b2-vaultwarden-backup`（只限該 bucket），restic 密碼在 `restic-vaultwarden-backup`，同時在 oracle `/etc/vaultwarden-backup.pass`。2026-10-20 首次備份並還原驗證：`PRAGMA integrity_check` = ok。
- 把 secret 寫到遠端檔案而不經過對話：`vault run K=item#username S=item -- sh -c 'printf ... "$K" "$S" | ssh host "sudo sh -c \"umask 077; cat > /etc/x.env\""'`。
