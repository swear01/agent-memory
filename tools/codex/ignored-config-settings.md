---
title: Codex ignored MCP type and legacy feature settings
scope: Codex configuration and transfer_MAC fleet MCP rendering
status: verified_with_windows_pending
updated: 2026-10-03
---

## Root cause and fix

Codex CLI 0.159.2 and the Mac app-bundled 0.159.0-alpha.12.1 ignore `features.rmcp_client` and `mcp_servers.context7.type`. Remove the former legacy feature entry. Codex selects MCP transport from `command` or `url`, so its TOML adapter must omit the canonical `type` discriminator. Retain `type` in shared `servers.json` and other clients that use it.

The canonical `transfer_MAC` renderer is `scripts/sync-ai-agent-configs.py`. PR 56 merged as `dc6bd635d1d8af15a4a0b340a2a85caa25861aa8`; the renderer SHA-256 is `3fe9ec1b197224986ba94efe83f392011540249fe5a56768930a5e9758150d93`. Regression coverage checks that the rendered Context7 table has no `type`, preserves unrelated settings, supports dry-run, and is idempotent. All 115 repository tests and CI passed; Swear Review found no issues for the PR head.

The reported `[features.guardianv2].thread_context` deprecation was absent from the live configuration and effective config layers. Fresh app-server initialization did not reproduce it. Do not claim to have removed a field that was not present or infer that an already-open app has refreshed its displayed warning.

## Fleet deployment and verification

2026-10-03: Mac, mazu, athena, cthulhu, valkyrie, Zeus runner, and Oracle passed configuration readback, strict app-server initialization without config warnings, and Context7 STDIO initialization plus `tools/list`. The six Linux hosts use Codex 0.159.2. Their only removed setting was `mcp_servers.context7.type`; `rmcp_client` and Guardian `thread_context` were already absent.

Write once to the shared NFS home used by mazu, athena, cthulhu, and valkyrie, then verify every host independently. Zeus's HAPI runner uses the separate `su_zeus` home; the default `zeus` SSH account reads the other shared home and is insufficient for runner verification. Oracle has another independent home.

Linux deployed renderer: `<home>/.local/share/agent-config-sync/scripts/sync-ai-agent-configs.py`. Its default canonical MCP copy is under the same runtime root at `stow/agents/.agents/mcp/servers.json`; the existing live source remains `<home>/.agents/mcp/servers.json`. Re-rendering should use the live source explicitly:

```sh
python3 "$HOME/.local/share/agent-config-sync/scripts/sync-ai-agent-configs.py" --canonical "$HOME/.agents/mcp/servers.json" render-mcp --dry-run
```

The deployment backed up `config.toml` under `<home>/.cache/codex-config-20261003/`, compared parsed TOML against the original with only the ignored key removed, verified the renderer hash, and reran rendering to prove idempotence. No HAPI service restart was required; existing processes may retain previously loaded settings.

Windows `swop` remains pending. It was absent from the current HAPI online machine list; external SSH attempts timed out and the known internal SSH endpoint was unreachable from mazu. Do not reuse the older eight-host success snapshot as evidence for this rollout.

## Memory and QMD completion rule

Publish the portable Markdown note to shared memory, synchronize each independent checkout once, and refresh each host's local QMD index separately. Keep `QMD_CONFIG_DIR` and `XDG_CACHE_HOME` host-specific even when memory lives on NFS. Verify lexical retrieval and indexed-content readback after `qmd update` and `qmd embed`; Git sync or an index counter alone does not prove retrieval. Use GPU defaults first and apply a CPU fallback only after an actual embedding-context failure.
