---
title: HAPI updates and restarts must preserve child sessions and fail closed
scope: tools/hapi
project: hapi
status: active
confidence: high
evidence: Repeated rollout rehearsals, supervisor policy checks, exact-process recovery, cleanup ordering, platform verification, and a live Mac session-recovery check on 2026-08-20.
created: 2026-08-18
updated: 2026-08-20
tags:
  - hapi
  - supervisors
  - deployments
  - sessions
  - cleanup
source_refs:
  - public-release:supervisor-safe-operations
  - hapi-live:swairM5-session-recovery-2026-08-20
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

# Disconnected-session recovery on swairM5

For the Mac display name `swairM5`, treat an inactive session with `metadata.lifecycleState=running` as a disconnected-session candidate; do not bulk-revive archived sessions. Exclude the current session, then process each candidate in order:

1. `POST /api/sessions/:id/resume` with `{}`.
2. Use the returned `sessionId` (it may differ after merge) for `POST /api/sessions/:id/messages` with `{"text":"繼續","localId":"<fresh-uuid>"}`.
3. Verify `GET /api/sessions/:id` reports `active=true` and the message page contains that `localId`.
4. Check the exact process through `hapi runner list` and `hostPid`; do not infer active work from the database row alone.

`active=true` confirms a live session connection, not an actively generating turn; `thinking=false` means the agent is currently idle or waiting. A resumed session can run, emit agent messages, and later archive, so report those states separately. Keep the recovery namespace-scoped and retain no raw transcript or token in memory.

# Platform checks

Verify the loaded supervisor configuration, signing identity where applicable, protected-folder probes, and the platform's restart command after every rollout.
