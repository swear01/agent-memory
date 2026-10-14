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

# Current policy

The base policy defaults to autonomous execution for routine, reversible,
in-scope work. The agent owns the requested outcome end to end and continues
through implementation, verification, and safe in-scope fixes.

The approval boundary remains one Core Rule line: ask only for a concrete,
material problem—missing input, a user-owned decision, or an unrequested
destructive, irreversible, paid, or external action—and never skip a required
step for lack of approval; ask instead. Before asking, diagnose the issue and
complete safe, reversible, equivalent work. Necessary changes across files,
functions, tests, fixtures, configuration, dependencies, generated artifacts,
and documentation are in scope; unrelated refactors and features are not.

Equivalent tools and implementation methods may be substituted without asking
when they preserve behavior, security, data safety, verification strength,
target, and material cost. The agent must not return dummy results, skip
verification, hide failures, or silently downgrade scope.

# Verified behavior

The current base policy begins with:

```text
Default to autonomous execution: make and carry out the least-risky reasonable decisions for routine, reversible, in-scope work without asking.
```

On 2026-09-03, the published base change was propagated across the configured
Skillshare targets and fleet hosts. Removing the nonexistent
`cthulhu:<remote-home>/ICCAD2026B` entry left 10 valid managed repositories;
`agents_rule update-all` then completed successfully.

QMD memory is a reminder, not an event hook. The durable source of truth for
synchronization is the base template plus `managed-repos.txt`; future agents
should search this memory and follow the agent-rules skill when changing the
template. After changing this note, refresh QMD and verify retrieval with an
exact lexical query plus a structured query and source readback.

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

The user-approved concise rule is:

```text
Auto-push and verify task commits; sync memory via ~/agent-memory, then refresh and verify QMD.
```

For memory synchronization, inspect the configured shared-memory repository;
the local Codex note directory lacking a remote is not a synchronization blocker.
QMD indexing, Git publication and fleet deployment are distinct outcomes.

The 2026-10-14 deployment snapshot verified the shortened rule and all five
agent links on Mac, mazu, cthulhu, valkyrie, zeus and oracle. Athena's shared
source was updated but its SSH endpoint refused connections, so direct host
verification remained pending. Swop was not updated: external SSH refused
connections and its last known LAN endpoint timed out. Do not turn this
snapshot into an all-host completion claim; recheck reachability before retrying.
