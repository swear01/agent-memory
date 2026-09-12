---
title: /apps CIC 腳本不相容 bash nounset
scope: machines/mazu
machine: mazu
status: active
created: 2026-09-12
updated: 2026-09-12
tags:
  - apps
  - cic
  - eda
  - bash
---

# 結果

- `/apps/bin` launcher 若在 `set -u` 下直接 `source` Synopsys/Cadence CIC 腳本，會在 `LD_LIBRARY_PATH` 未設定時失敗：`unbound variable`。CIC 寫成 `if [ -z "$LD_LIBRARY_PATH" ]`。
- 同一個 `source` 接著會載入對應的 `license.sh`，同樣用 `if [ -z "$LM_LICENSE_FILE" ]`，nounset 一樣炸。不要 cat `license.sh`，也不要記錄其值。
- 實測失敗點：`synopsys/CIC/vcs.sh`、`verdi.sh`、`spyglass.sh`、`synthesis.sh`；`cadence/CIC/innovus.sh`、`jasper.sh`。
- 作法：launcher 維持 `set -euo pipefail`，但在 `source` 前後 `set +u` / `set -u`。不要為了這件事去改 CIC 或 `license.sh`。
- `synthesis.sh` 會把 `.../amd64/syn/bin` 加進 `PATH`，source 後 `command -v dc_shell` 可用。`innovus.sh` / `jasper.sh` 同樣會把 `innovus` / `jg` 放進 `PATH`。
- `env.sh` 只 source 兩個 `license.sh`，不要 source 完整 tool CIC。在沒有 `set -u` 的一般 bash 下，`test -n "$LM_LICENSE_FILE"` 會成功；不要把變數值寫進筆記。
