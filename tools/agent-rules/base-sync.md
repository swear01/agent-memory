---
title: agent-rules base synchronizes managed AGENTS.md files
scope: tools/agent-rules
tool: agent-rules
status: active
confidence: high
created: 2026-08-20
updated: 2026-10-14
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

After changing `templates/base.md`, publish the canonical Skillshare source,
sync each independent fleet source, then run:

```bash
~/.agents/skills/agent-rules/agents_rule update-all
```

`update-all` is the final managed-repository propagation step. It updates the
`agents_rule-base` block in every listed repository and does not add or remove
entries from the list.

The local managed-repository list is `~/.agents/managed-repos.txt`. It may include
the current user's playground and global agent directory, plus other explicitly
managed repositories on that machine.

`managed-repos.txt` is machine-local and is not synchronized through this memory
repository; register the appropriate repositories separately on another machine.

If a listed local or remote repository no longer exists, remove only that stale
line from `managed-repos.txt` and rerun `update-all`. A missing managed path can
otherwise fail the whole command even when the other entries are valid.

`agents_rule init --force` only creates or replaces the managed block. If an
existing `AGENTS.md` has an older unmanaged copy of the rule outside that block,
replace the stale wording separately; otherwise Pi still reads both versions.

# Global instructions and fleet deployment

The complete global instruction file has a separate canonical source:
`<transfer-MAC-root>/stow/agents/.agents/AGENTS.md`. On Mac, the global
`~/.agents/AGENTS.md` links to this file; Codex, Claude, Gemini, OpenCode and Pi
use links to the global file. The base template is in the shared-skills
submodule at `<transfer-MAC-root>/.skillshare/skills/agent-rules/templates/base.md`.

Rules outside `agents_rule-base`, including `Local Preferences and Tools`,
are also shared across agents and can be deployed across machines. They do not
have to move into the base merely to support fleet deployment. However,
`agents_rule update-all` replaces only the managed block and leaves these
outside rules untouched. Git push publishes the source; it does not apply the
new global rule on independent hosts. Deploy the changed rule separately,
preserve other host-specific content, then read it back and verify agent links
on each host. Write a shared NFS source once and verify every alias separately.

See the live base template for policy text. It can change independently of this
procedure. The 2026-09-03 and 2026-10-14 fleet readbacks are available with
`git show 06f50e3:tools/agent-rules/base-sync.md`.
