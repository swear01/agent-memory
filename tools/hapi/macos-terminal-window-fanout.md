---
title: HAPI Pi GUI automation must not fan out Terminal.app windows
scope: tools/hapi
project: hapi
status: active
confidence: high
evidence: Read-only audit of a completed macOS HAPI Pi session on 2026-08-20; Terminal.app tab TTYs and session commands matched exactly.
created: 2026-08-20
updated: 2026-08-20
tags:
  - hapi
  - macos
  - terminal
  - pi
  - gui-automation
redaction: passed
generated_by: openai-codex/gpt-5.6-luna
---

# Verified root cause

A HAPI-started Pi session handling Visual Maimai and macOS Full Disk Access repeatedly ran:

```text
tell application "Terminal" to do script ...
```

without targeting an existing window. macOS Terminal opened a new tab/window for each invocation. The command then exited but left the login shell alive, so the user saw many blank `login-zsh` windows. This was agent behavior, not HAPI's normal Bun PTY terminal manager.

# Prevention rule

For future HAPI/Pi GUI tasks:

- Prefer the existing HAPI PTY or direct `osascript`/`open` execution; do not use Terminal.app as an accessibility bridge by default.
- If Terminal.app is genuinely required, reuse one explicitly identified tab/window and close it after the command completes.
- Never issue repeated `tell application "Terminal" to do script` calls without an explicit cleanup plan or user confirmation.
- Before leaving a GUI task, verify that no helper Terminal tabs remain and that the agent session is archived/stopped when the work is complete.
