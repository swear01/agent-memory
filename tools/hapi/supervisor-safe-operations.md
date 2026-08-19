---
title: HAPI updates and restarts must preserve child sessions and fail closed
scope: tools/hapi
project: hapi
status: active
confidence: high
evidence: Repeated rollout rehearsals, supervisor policy checks, exact-process recovery, cleanup ordering, and platform verification.
created: 2026-08-18
updated: 2026-08-19
tags:
  - hapi
  - supervisors
  - deployments
  - sessions
  - cleanup
source_refs:
  - public-release:supervisor-safe-operations
redaction: passed
generated_by: openai-codex/gpt-5.6-luna
---

# Restart contract

Restart only the supervisor's main runner:

- Linux systemd: `KillMode=process`
- macOS launchd: `AbandonProcessGroup=true`
- PM2: `treekill=false`

Do not manually kill runner-created children. Do not use broad process matchers.

# Handoff recovery

If a version handoff leaves a duplicate main runner, read the state PID, verify its command line is exactly the expected main `hapi runner start-sync` process, and stop only that PID.

After recovery, verify supervisor `MainPID`, state PID, restart stability, API identity, and the pre-update child-session baseline.

# Update safety

Use `set -o pipefail`; nested update or restart failures must propagate non-zero. Write installer downloads to a user cache, not a shared or privileged temporary directory.

On shared-filesystem hosts, publish a shared binary once, then verify each runner before restarting the hub.

# Cleanup safety

Operate only on the exact registered machine identity and back up the hub database first. For connected sessions:

1. Archive or terminate through the Hub.
2. Use the runner `stop-session` endpoint.
3. Send TERM to an exact PID only after confirming the process is disconnected.

Directly killing a connected orphan can trigger CLI cleanup and create misleading hub sessions.

# Platform checks

Verify the loaded supervisor configuration, signing identity where applicable, protected-folder probes, and the platform's restart command after every rollout.
