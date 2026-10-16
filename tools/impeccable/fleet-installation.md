---
title: Impeccable installation, hooks, and concise UI policy
scope: tools/impeccable
status: active
updated: 2026-10-16
---

2026-10-16 verified snapshot; recheck host availability and PR state before further deployment.

## Installed scope

- Mac, mazu, cthulhu, valkyrie, Zeus (`su_zeus`), Oracle: full Impeccable 4.6.0 skill, engine 0.1.15, global UI rule, and enabled hooks. Codex native `skills/list` discovered the skill; `hooks/list` confirmed both Impeccable handlers trusted, with no hook errors. Edit/Stop engine smoke checks passed; backend-only edits stayed silent.
- Athena and Windows swop were offline and remain pending. No active runner or session was restarted.
- Upstream tag `skill-v4.6.0`; installed shared-skills commit `b048ea108e2f8082d93a588ab0abb29f00f57535`. Skill payload and installed state are separate from PR merge status.

## Ownership and use

Skillshare owns the portable skill and target links. Mac canonical source is `<transfer-MAC-root>/.skillshare/skills`; remote source is `<remote-home>/.config/skillshare/skills`. Run `skillshare sync` after source changes; preserve independent local skills.

`<transfer-MAC-root>/scripts/sync-ai-agent-configs.py` renders hooks from `stow/agents/.agents/impeccable-hooks.json`. Claude has SessionStart, PostToolUse (`Edit|Write`), and Stop; Codex has PostToolUse (`Edit|Write|apply_patch`) and Stop. Existing unrelated handlers/settings are preserved. Trust only the reviewed Impeccable handlers through native Codex controls; do not enable a global trust bypass or trust unrelated modified hooks.

Normal frontend prompts can select Impeccable automatically; users need not paste the policy each time. To request the workflow explicitly: `Use Impeccable distill on this UI.` Hooks provide automatic checks, not a guarantee of good copy. The common AGENTS rule requires distill on changed UI before delivery.

The user prefers concise English global rules and no helper text by default. Keep copy only when it helps the current action, prevents a concrete error, or explains an important consequence; preserve existing style, functionality, accessibility, and essential warnings.

Shorter proposed wording below was discussed but **not applied**; the deployed rule remains the earlier longer English version:

> Default to no helper text. Before delivery, run Impeccable distill on changed UI; preserve existing style, functionality, accessibility, and essential warnings.

## Verification boundaries and follow-up

- transfer_MAC PR #60 merged at `3777a4efdd3b41da17cc6c331e10c4b87d00a99d`; shared-skills PR #47 remains open at the installed commit, with unresolved upstream live-overlay review findings and no passing provider. Rechecked both PR states on 2026-10-16. Do not treat a completed review job as a clean review.
- Full live browser workflow and native Windows execution remain unverified. A Chromium modal probe timed out; an accidental pass claim was withdrawn. Never cite it as successful evidence.
- Skillshare's automatic audit flagged vendored code; reviewed import proceeded after manual triage. Do not report the automatic audit as passing.
- A shorter global-rule edit does not require reinstalling the engine, altering hooks, or trusting them again. Replace only the preference line outside `agents_rule-base`; preserve host-specific settings.
