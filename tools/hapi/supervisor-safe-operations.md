---
title: HAPI updates and restarts must preserve child sessions and fail closed
scope: tools/hapi
project: hapi
status: active
confidence: high
evidence: Repeated live rollouts, supervisor policy checks, exact-PID recovery, cleanup ordering, and platform-specific verification.
created: 2026-08-18
updated: 2026-08-18
tags:
  - hapi
  - supervisors
  - deployments
  - sessions
  - cleanup
source_refs:
  - pilot-hapi-gist:hapi-gist.md:160-186
  - pilot-hapi-gist:hapi-gist.md:227-233
  - pilot-hapi-gist:hapi-gist.md:253-255
  - pilot-hapi-gist:hapi-gist.md:373-382
  - pilot-hapi-gist:hapi-gist.md:464-490
  - pilot-hapi-gist:hapi-gist.md:580
  - pilot-hapi-gist:hapi-gist.md:648
generated_by: openai-codex/gpt-5.6-luna
redaction: passed
---

# Restart contract

Restart only the supervisor's main runner:

- Linux systemd: `KillMode=process`
- macOS launchd: `AbandonProcessGroup=true`
- Oracle PM2: `treekill=false`

Do not manually kill `--started-by runner` children. Do not use broad matchers such as `" runner "` or `pgrep -f "hapi runner"`.

# Handoff recovery

If a version handoff leaves a duplicate main runner, read the machine-specific state PID, verify its command line is exactly the main `hapi runner start-sync` process and does not contain `--started-by runner`, then stop only that PID.

After recovery, verify supervisor `MainPID`, state PID, restart stability, API identity, and the pre-update child-session baseline.

# Update safety

Use `set -o pipefail`; nested update or restart failures must propagate non-zero. Write installer downloads to a user cache such as `~/.cache/hapi-install-tmp`, not root `/tmp`.

On shared NFS hosts, write a shared binary once, then verify each runner before restarting the hub.

# Cleanup safety

Operate only on the exact machine ID and back up the hub DB first. For connected sessions:

1. Archive or terminate through Hub.
2. Use the runner `stop-session` endpoint.
3. Send TERM to an exact PID only after confirming the process is disconnected.

Directly killing a connected orphan can trigger CLI cleanup and create misleading hub sessions.

# Platform checks

For macOS, verify the loaded LaunchAgent's `NumberOfFiles=65536`, stable signing identity, and protected-folder probes. For Oracle, replace the binary while the old runner remains alive, use `pm2 restart hapi --no-treekill --update-env`, then run `pm2 save`.
