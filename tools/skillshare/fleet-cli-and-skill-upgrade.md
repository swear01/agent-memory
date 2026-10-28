---
title: "Skillshare：fleet 升級 CLI 與內建 skillshare skill"
scope: tools/skillshare
tool: skillshare
status: active
confidence: high
evidence: 2026-10-19 從 v0.20.25 升到 v0.25.13；2026-10-25 全部升到 v0.26.4；2026-10-28 升到 v0.27.0（Mac, Mazu, Cthulhu, Valkyrie, Oracle；Zeus 依指示排除 swear02，Athena 送修中排除），逐台以 `skillshare version`、`skillshare pull` 與 source HEAD 驗證。
created: 2026-10-19
updated: 2026-10-28
tags:
  - skillshare
  - fleet
  - upgrade
---

# Skillshare fleet 升級

CLI 與內建的 `skillshare` skill 是兩件事，要分開升級、分開驗證。`skillshare doctor` 會分別回報 `CLI` 與 `skill_version`。

## CLI

- Mac 透過 Homebrew 安裝：`brew upgrade skillshare`。`status` 顯示的「可升級版本」可能落後；2026-10-19 提示是 0.25.8，實際裝到的是 v0.25.13。
- Linux 主機是放在 `<remote-home>/.local/bin/skillshare` 的獨立 binary。非互動 SSH 要先 `export PATH=$HOME/.local/bin:$PATH`，再執行 `skillshare upgrade --cli --force`（只升 CLI；不加 `--cli` 會連 skill 一起升）。
- mazu、cthulhu、valkyrie 看到的是同一個 `swear01` home 裡的 binary：在 mazu 升一次後，三台都回報新版。zeus（`su_zeus`，`swear02`）與 oracle（`ubuntu`，aarch64）要各自升級。
- `skillshare version` 會印 ASCII banner，取版本號用 `skillshare version 2>&1 | grep -o "v[0-9.]*" | tail -1`。
- Windows swop 用 `skillshare upgrade --cli --force` 升級。2026-10-25 從 0.20.27 升到 v0.26.4；它的 git pull 卡住問題見 [[git-ssh-portablegit]]。Athena 依 hapi skill 政策排除。
- 2026-10-25 GitHub 最新 release 是 v0.26.4，沒有 0.30 系列，不要和 HAPI 的版本號（0.30.x）混淆。

## 內建 skill

內建 skill 存在 `shared-skills` source（`transfer_MAC` 的 submodule `.skillshare/skills`）。這個 skill 的歷史只有上游版本更新，沒有本地修改，所以可以直接覆蓋：

1. `skillshare upgrade --skill --dry-run`，接著執行 `skillshare upgrade --skill --force`。會改到 `skillshare/` 與 `.metadata.json`；後者也可能出現其他項目 key 的重新排序，屬於無害變更。
2. `skillshare audit skillshare --format json` 確認結果 clean。
3. Mac 的 submodule 依慣例保持 detached HEAD。若 HEAD 等於 `origin/main`，先 `git checkout -B main origin/main`，再執行 `skillshare push --dry-run` 與 `skillshare push -m ...`。push 完執行 `git checkout --detach <新 SHA>`，回到 detached 狀態。Mac 上不要用 `skillshare pull`，原因見 `projects/transfer_MAC/skillshare.md`。
4. superproject 只 `git add .skillshare/skills` 並提交指標，不要碰其他還沒提交的 dotfile 修改。
5. 在 mazu、zeus、oracle 各執行一次 `skillshare pull`（mazu 這次 pull 也涵蓋 cthulhu、valkyrie），再逐台核對 source `rev-parse HEAD` 與 `skillshare/SKILL.md` 的 `version:`。本機改完後也要 `skillshare sync`。

相關筆記：[[locally-maintained-skill]]（自行維護的 skill 不能這樣覆蓋）。
