---
title: DVLab CRA 視覺 parity PR #101
scope: projects/dvlab-website
project: DVLab-NTU/dvlab-ntu.github.io
status: active
updated: 2026-09-30
---

## PR 與發布邊界

- Production 目前是 Astro on `main`；primary 為 `https://dvlab.ee.ntu.edu.tw`（inari），GitHub Pages 是 backup。PR #100（包含 Host page 等內容）已合併；其 main SHA 約為 `7f1a78e`。Inari release 命名模式為 `releases/<date>-host-<shortsha>`。
- PR #101 仍是 open draft：`https://github.com/DVLab-NTU/dvlab-ntu.github.io/pull/101`，branch 為 `cursor/cra-visual-parity-5109`，cloud agent 為 `bc-0540771a-7b2d-5206-813f-a73aab7c5109`。截至 2026-09-30 早上，最新 visual commit 是 `dad9355d2a328580ac147500f5b61fc011501315`。
- **在使用者 review 前不要 merge PR #101。** Merge 後 deploy target 仍同時是 GitHub Pages 與 inari；PR merge 或 Pages 成功本身不代表 inari 已更新。

## 使用者指定的視覺方向

- 保留 Astro，向舊 CRA（commit `6e33b72`）restyle；目前先從 nav/routes 移除 Life。
- Host photo 使用原始 CRA `ric.jpeg`，原 repo path 是 `frontend/public/assets/images/examples/host-profile/ric.jpeg`；不要新增重複的 PI badges。
- Navbar 使用 liquid-glass / frosted translucent 與 `backdrop-filter`；header/body 的 `padding-top` 透過 ResizeObserver 追蹤 `--site-header-height`。
- Members 使用 horizontal scrolling tracks/carousel；目前只有三個 categories：形式化驗證（`formal`/`ai-formal`/`verification`）、EDA/3DIC（`eda`/`3dic`/`architecture`）、Quantum。PI 不列入 categories。
- Body 使用 CRA 的 Helvetica Neue（weight 300），titles 使用 Coolvetica；字型放在 `public/fonts/cra/`。
