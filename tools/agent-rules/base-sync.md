---
title: agent-rules base synchronizes managed AGENTS.md files
scope: tools/agent-rules
tool: agent-rules
status: active
confidence: high
created: 2026-08-20
updated: 2026-08-20
tags:
  - agent-rules
  - AGENTS.md
  - synchronization
---

# Canonical source

The shared base policy lives at:

```text
~/.config/skillshare/skills/agent-rules/templates/base.md
```

The installed path `~/.agents/skills/agent-rules/` is a symlink to that skill source.

# Synchronization

After changing `templates/base.md`, run:

```bash
~/.agents/skills/agent-rules/agents_rule update-all
```

The local managed-repository list is `~/.agents/managed-repos.txt`. It may include
the current user's playground and global agent directory, plus other explicitly
managed repositories on that machine.

`managed-repos.txt` is machine-local and is not synchronized through this memory
repository; register the appropriate repositories separately on another machine.

# Verified behavior

The base rule for ambiguity is currently:

```text
Execute ONLY what was requested. Clarify only when necessary.
```

QMD memory is a reminder, not an event hook. The durable source of truth for
synchronization is the base template plus `managed-repos.txt`; future agents
should search this memory and follow the agent-rules skill when changing the
template.
