---
title: "Skillshare：fleet 升級 CLI 與內建 skillshare skill"
scope: tools/skillshare
tool: skillshare
status: active
confidence: high
created: 2026-10-19
updated: 2026-10-28
tags:
  - skillshare
  - fleet
  - upgrade
---

## 分開驗證 CLI 與內建 skill

`skillshare doctor` 分別回報 CLI 和 `skill_version`，兩者各自更新。顯示的可升級版本曾落後於實際 release；讀實際 binary、package source、`skillshare version` 和 skill 的版本欄位。

- Mac 用 Homebrew 管 CLI。Linux 登入後沿 `PATH` 查實際 executable，非互動 SSH 檢查 `~/.local/bin`。若現行版本支援，`skillshare upgrade --cli` 限定 CLI；`--force` 前先查管理來源及覆寫範圍。
- 寫入共享 NFS home 前，查各 host 的同一註冊帳號、mount 與實際 binary 是否共用，對每份獨立安裝只改一次，再逐台讀回。Athena 是否在維護範圍，以 HAPI 現行 handover 為準，舊的送修排除只屬當時批次。
- Windows CLI 的 Git／SSH 與 noninteractive PATH 問題見 [swop CLI、認證與 QMD](../../machines/swop/cli-path-auth-and-qmd.md)。使用中的 Git binary、SSH alias 和 key chain 要隨各環境確認。

## 內建 skill

`shared-skills` 是 transfer_MAC 的 `.skillshare/skills` submodule。先確認內建 `skillshare/` 沒有本地改動，再用 `skillshare upgrade --skill --dry-run` 檢查；經授權才執行更新。若該 skill [已有本地維護](locally-maintained-skill.md)，沿其來源更新。版本、`.metadata.json` 和 audit 分開核對，格式排序變化不代表其他 skill 升級。

Mac source 採 detached submodule；依 [transfer_MAC Skillshare 維運](../../projects/transfer_MAC/skillshare.md) 驗證來源與發布，合併後更新 superproject 指標及 host target。各獨立 source 核對 `rev-parse HEAD` 和實際 target 的 skill header，依 source→target 流程執行 `skillshare sync`，保留其他未提交 dotfiles。

[原始部署批次與驗證](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/skillshare/fleet-cli-and-skill-upgrade.md)
