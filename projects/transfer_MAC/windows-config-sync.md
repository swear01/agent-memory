---
title: sync-ai-agent-configs 在 Windows 上需使用 POSIX 路徑與 callable re.sub
scope: projects/transfer_MAC
project: transfer_MAC
tool: Codex
status: active
confidence: high
created: 2026-09-28
updated: 2026-09-28
tags:
  - transfer-mac
  - codex
  - toml
  - windows
  - mcp
---

# 問題現象

在 Windows 主機（`swop`）上，當執行 `sync-ai-agent-configs.py render-mcp` 或 `render` 同步 MCP 設定至 `~/.codex/config.toml` 後，Codex CLI 與 HAPI runner 啟動時崩潰並拋出錯誤：

```text
Error loading config.toml:
<home>/.codex/config.toml:148:16: too few unicode value digits, expected unicode hexadecimal value
```

`codex doctor` 報告配置無法載入（invalid configuration），且 HAPI runner 無法獲取 Codex 模型清單而中斷連線。

# 根因分析

1. **`re.sub` 字串替換時的反斜杠吞噬**：
   在 `update_codex_config` 與 `update_toml_string` 中，原本使用 `pattern.sub(block, original)`。在 Python 的 `re.sub` 實作中，若 `repl` 為字串，正則引擎會將其中的反斜杠視為轉義序列處理，使得原本經 JSON 序列化轉義的 `\\`（如 `C:\\Users\\...`）被還原為單一反斜杠 `\`（`C:\Users\...`）。
2. **TOML `\U` Unicode 轉義解析失敗**：
   TOML 雙引號字串（basic string）將 `\U` 視为 8 碼十六進制 Unicode 逸出碼（`\U00000000`）。由於路徑中為 `\Users`，接續的字元並非有效 16 進位數，TOML 解析器判定語法錯誤直接終止。
3. **路徑反斜杠混用**：
   `expand_home` 函式原本使用 `str(home)` 替換 `${HOME}`，在 Windows 上會把反斜杠注入到原本使用 POSIX 斜杠的相對路徑中（例如 `${HOME}/.npm-global/bin/...` 變成 `C:\Users\<user>/.npm-global/bin/...`）。

# 修復方案

1. **改用 `home.as_posix()`**：
   在 `expand_home` 中將 `value.replace("${HOME}", str(home))` 改為 `value.replace("${HOME}", home.as_posix())`。Windows 核心與大多數跨平台工具皆支援正斜杠路徑，可徹底避免在 TOML、JSON、YAML 等配置中產生轉義歧義。
2. **`pattern.sub` 改用 callable**：
   將所有區塊替換改用 lambda 形式：
   - `pattern.sub(lambda _: block, original)`
   - `pattern.sub(lambda _: replacement, preamble, count=1)`
   當 `repl` 為函式時，Python 正則引擎不會對傳回字串進行逸出碼二次解析，保證內容原樣寫入。

# 驗證

- 重新執行 `python sync-ai-agent-configs.py render-mcp`，寫入的 `config.toml` 包含 `command = "<home>/.npm-global/bin/context7-mcp"`，格式合法。
- 執行 `codex doctor` 確認配置正常載入（`config.toml parse ok`），CLI 正常回應。
