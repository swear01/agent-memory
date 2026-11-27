---
title: Codex ignored MCP type and legacy feature settings
scope: Codex configuration and transfer_MAC fleet MCP rendering
status: active
updated: 2026-10-03
---

## Root cause and fix

Codex CLI 0.159.2 and the Mac app-bundled 0.159.0-alpha.12.1 ignore `features.rmcp_client` and `mcp_servers.context7.type`. Remove the former legacy feature entry. Codex selects MCP transport from `command` or `url`, so its TOML adapter must omit the canonical `type` discriminator. Retain `type` in shared `servers.json` and other clients that use it.

The canonical `transfer_MAC` renderer is `scripts/sync-ai-agent-configs.py`. See transfer_MAC PR #56 for the renderer change. Regression coverage checks that the rendered Context7 table has no `type`, preserves unrelated settings, supports dry-run, and is idempotent.

The reported `[features.guardianv2].thread_context` deprecation was absent from the live configuration and effective config layers. Fresh app-server initialization did not reproduce it. Do not claim to have removed a field that was not present or infer that an already-open app has refreshed its displayed warning.

## Fleet deployment and verification

2026-10-03: Mac, mazu, athena, cthulhu, valkyrie, Zeus runner, and Oracle passed configuration readback, strict app-server initialization without config warnings, and Context7 STDIO initialization plus `tools/list`. The six Linux hosts use Codex 0.159.2. Their only removed setting was `mcp_servers.context7.type`; `rmcp_client` and Guardian `thread_context` were already absent.

Write once to the shared NFS home used by mazu, athena, cthulhu, and valkyrie, then verify every host independently. Zeus's HAPI runner uses the separate `su_zeus` home; the default `zeus` SSH account reads the other shared home and is insufficient for runner verification. Oracle has another independent home.

Linux deployed renderer: `<home>/.local/share/agent-config-sync/scripts/sync-ai-agent-configs.py`. Its default canonical MCP copy is under the same runtime root at `stow/agents/.agents/mcp/servers.json`; the existing live source remains `<home>/.agents/mcp/servers.json`. Re-rendering should use the live source explicitly:

```sh
python3 "$HOME/.local/share/agent-config-sync/scripts/sync-ai-agent-configs.py" --canonical "$HOME/.agents/mcp/servers.json" render-mcp --dry-run
```

The deployment backed up `config.toml` under `<home>/.cache/codex-config-20261003/`, compared parsed TOML against the original with only the ignored key removed, verified the renderer hash, and reran rendering to prove idempotence. No HAPI service restart was required; existing processes may retain previously loaded settings.

Windows was outside the 2026-10-03 verification. Check the current fleet handover and read back the active Windows renderer and native client before reporting success. An earlier rollout or current runner liveness cannot establish those checks.

Deployment hashes and the complete 2026-10-03 verification are available with `git show 54da0f0adfdb3dacb3bca59a729b12975d44ec9b:tools/codex/ignored-config-settings.md`. For publishing and refreshing shared memory, see [shared-memory workflow](../../global/workflows/shared-memory.md).
