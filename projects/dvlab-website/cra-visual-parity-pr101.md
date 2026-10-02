---
title: DVLab CRA 視覺 parity PR #101
scope: projects/dvlab-website
project: DVLab-NTU/dvlab-ntu.github.io
status: active
updated: 2026-10-02
---

## Deployed 2026-10-01

- PR #101 squash-merged to `main` as `385e247`.
- GitHub Pages + inari both served that build; inari then advanced for PR #102 (see `typewriter-mottos-pr102.md`). Snapshot at #101: `current` → `releases/20261001-385e247` (previous `20260930-host-7f1a78e` kept).
- Do not treat older draft / “do not merge” notes below as current status.

## PR 與發布邊界

- Production 目前是 Astro on `main`；primary 為 `https://dvlab.ee.ntu.edu.tw`（inari），GitHub Pages 是 backup。
- **PR #101 已 merge 並雙站部署（2026-10-01，`385e247` / inari `20261001-385e247`）。** 之後更新仍須分別驗證 Pages 與 inari；Pages 成功本身不代表 inari 已更新。
- **PR #102**（typewriter mottos）已於同日 merge+deploy：`2d370c3` / inari `20261001-2d370c3` — 詳見 `typewriter-mottos-pr102.md`。不要留下「do not merge #102」舊註記。

## 使用者指定的視覺方向

- 保留 Astro，向舊 CRA（commit `6e33b72`）restyle；目前先從 nav/routes 移除 Life。
- Host photo 使用原始 CRA `ric.jpeg`，原 repo path 是 `frontend/public/assets/images/examples/host-profile/ric.jpeg`；不要新增重複的 PI badges。
- Navbar 使用 liquid-glass / frosted translucent 與 `backdrop-filter`；header/body 的 `padding-top` 透過 ResizeObserver 追蹤 `--site-header-height`。
- Members 使用 horizontal scrolling tracks/carousel；目前只有三個 categories：形式化驗證（`formal`/`ai-formal`/`verification`）、EDA/3DIC（`eda`/`3dic`/`architecture`）、Quantum。PI 不列入 categories。
- Body 使用 CRA 的 Helvetica Neue（weight 300），titles 使用 Coolvetica；字型放在 `public/fonts/cra/`。
- Homepage CRA culture typewriter mottos：`#lab-introduction`（PR #102）。
- Liquid-glass chips / panels（PR #103，deploy `9aff7ee` / inari `20261002-9aff7ee`）：`--glass-bg*` 核准半透明；news date chips、paper list denser panel、paper detail 會議+年份 labeled chips、unified footer。詳見 `liquid-glass-chips-pr103.md`。
