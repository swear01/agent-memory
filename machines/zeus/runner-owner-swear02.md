---
title: Zeus 的 HAPI runner 屬於 swear02，swear01 的 hapi-runner unit 是失效殘留
scope: machines/zeus
machine: zeus
status: active
created: 2026-10-26
updated: 2026-10-26
tags:
  - hapi
  - runner
  - systemd
---

- 檢查 zeus 的 runner、session 程序、版本、`runner.state.json`、`/proc/<pid>/exe` 一律用 `ssh su_zeus`（swear02）。用 `ssh zeus`（swear01）看不到 swear02 的 `/proc/<pid>/exe`，也讀不了 `/var/tmp/hapi-zeus/runner.state.json`，掃描 session 會得到空結果而被誤判成「沒有舊 session」。
- swear01 的 user unit `hapi-runner.service` 自 2026-10-22 起持續 `activating (auto-restart)`：`Failed to load environment files: Permission denied`，因為 `EnvironmentFile=/var/tmp/hapi-runner/runner.env` 屬於 swear02 且為 600。它不是實際的 runner，不影響 swear02 的 runner；`hapi machines` 的 zeus 在線是 swear02 的 runner。
- 2026-10-26 以 `su_zeus` 驗證：MainPID 與 `runner.state.json` 的 pid 一致（1359242），`startedWithCliVersion` 為 0.30.7.7，NRestarts 為 0，`/var/tmp/hapi-zeus/runtime/` 只有 0.30.7.7。
