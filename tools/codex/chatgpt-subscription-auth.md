---
title: Codex fleet 固定使用 ChatGPT 訂閱登入
scope: tools/codex
status: active
updated: 2026-10-17
---

# Canonical 設定

`~/.codex/config.toml` 頂層設定：

```toml
forced_login_method = "chatgpt"
```

使用已存在的 transfer_MAC renderer 管理；先發現實際 checkout，再從其根目錄執行：

```sh
python3 scripts/sync-ai-agent-configs.py render-codex-auth --dry-run
python3 scripts/sync-ai-agent-configs.py render-codex-auth
codex login status
```

完整 `render` 同樣會套用。獨立 `render-codex-auth` 只改登入欄位，保留模型、MCP、profiles 與專案設定；不另建 Codex wrapper。部署前須已有 ChatGPT 登入，這個欄位不會代替使用者登入。此次各獨立 home 保留 `~/.codex/backups/config.toml.before-chatgpt-auth-20261017`，權限 `0600`。

# 2026-10-17 已驗證範圍與限制

- Mac、Mazu、Cthulhu、Valkyrie、Zeus、Oracle 皆已設定，且真實 `codex login status` 仍為 ChatGPT。Mazu 共用 home 只寫一次，再逐台讀回。
- 六台都用臨時 `CODEX_HOME`、假金鑰及 file credential store 隔離測試；`codex login --with-api-key` 與 `CODEX_API_KEY` 的 `codex exec` 均 exit 1 拒絕啟動，沒有建立 auth.json。測試未碰真實憑證或送模型請求。
- 新啟動程序才載入設定；既有 Codex／App 需重開。Athena [送修排除](../../machines/athena/repair-status.md)；Windows swop 依使用者指示略過，未部署或驗證。
- 這是使用者預設設定，CLI `-c`、profiles、不同 `CODEX_HOME` 可覆寫。需要管理員限制時另用 system requirements 的 `allowed_login_methods = ["chatgpt"]`，本次未部署管理員政策。

來源：transfer_MAC PR #65（merge `5f37d6fe44b478952fdb891aef446872c3094f6a`），121 項測試與 CI 通過；repo `docs/mac/40_AI_Agent_MCP_and_Skills.md` 記錄此政策。官方依據：<https://developers.openai.com/codex/auth/#enforce-a-login-method-or-workspace>。
