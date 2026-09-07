---
title: GitHub transport timeout 可用保留主機驗證的 SSH 443
scope: tools/git
tool: git
machine: mazu
status: verified
updated: 2026-09-07
---

Mazu 的 HTTPS Git fetch/ls-remote 與經 SSH 22 的 Git fetch 在限時內沒有完成，
但 GitHub API 與獨立 SSH 22 authentication 都成功。這不足以診斷防火牆或 NFS
根因，也不能稱 SSH 22 無法連線；不要直接重裝 Git、改憑證或全域 remote。

本次實測有效替代是 invocation-only SSH 443：remote URL 使用
`ssh://<ssh-user>@ssh.github.com/<owner>/<repo>.git`，Git 加上
`-c core.sshCommand='ssh -o BatchMode=yes -o ConnectTimeout=8 -o HostKeyAlias=github.com -p 443'`。
`HostKeyAlias=github.com` 使用現有已信任的 GitHub host key；沒有關閉 host-key
verification。第一次未指定 alias 時主機驗證失敗，保留該錯誤並修正主機對應，
不要使用 `StrictHostKeyChecking=no`。

主 repo fetch 與 Wiki fetch/push 都已成功；原 remote/global config 未改。
fetch/push 後仍需獨立核对精確 remote SHA，不能只憑指令回傳成功。
