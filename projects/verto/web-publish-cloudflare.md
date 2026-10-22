---
title: Verto Web 公開試玩：Cloudflare Pages 手動發布與 Unity build 縮小
scope: projects/verto
status: verified
updated: 2026-10-22
tags: [Verto, PHASE, Unity, WebGL, Cloudflare-Pages, wrangler, cloudflared, build-size]
---

## 為何不像 hexapharma 用 GitHub Pages 自動部署

- hexapharma 是 npm 前端，GitHub-hosted runner 就能 build。Verto 的 Unity Web build 需要有授權的 Unity 6000.4.7f1 GUI Editor + Pipeline（專案規定不得改用 `-batchmode`），GitHub-hosted runner 沒有。
- 唯一自動方案是 self-hosted Mac runner；但 public repo 掛 self-hosted runner 時，fork PR 可以帶自己的 workflow 在 Mac 上執行（repo 預設 fork 核准政策只到 `first_time_contributors`）。安裝 runner（LaunchAgent）也被 auto mode classifier 以 Unauthorized Persistence 擋下。
- 使用者選擇本地 build 後手動上傳 Cloudflare Pages；GitHub 不會部署。2026-10-22 起專案規則（`AGENTS.md`，`CLAUDE.md` 是它的 symlink；Verto PR #12）要求：合併會改到遊戲（`Assets/`、`Packages/`、`ProjectSettings/`）的 PR 後，由合併的 session 在最新 `origin/main` 的全新 checkout 跑 `BuildWeb.py`、帶 `--commit-hash` 部署、再對線上跑 `WebLoadSmoke.cjs`；`npx wrangler pages deployment list --project-name verto` 的 Source 欄就是線上 commit，已是同一或更新的 commit 就略過。舊的 `web.yml`（等 runner、每次 push 都失敗）已移除（Verto PR #4，2026-10-22 合併為 `acab4ef`）。部署時從最新 `origin/main` 的 checkout 執行並帶 `--commit-hash`，避免舊 build 蓋掉新 build；Wrangler 會在 checkout 留下未追蹤的 `.wrangler/` 快取。

## Cloudflare Pages 設定（2026-10-21）

- 專案 `verto`，`*.pages.dev` 名稱為 `verto-6h5.pages.dev`；自訂網域 `https://verto.5334601.xyz/`，HTTP 自動 301 到 HTTPS。
- 發布：`npx wrangler pages deploy Builds/Web --project-name verto --branch main`，再跑 `node Verification/WebLoadSmoke.cjs https://verto.5334601.xyz/`。需先 `npx wrangler login`（瀏覽器 OAuth，使用者授權）。
- Wrangler 4.151 沒有 Pages 自訂網域指令。可行做法：用 Wrangler OAuth token（`~/Library/Preferences/.wrangler/config/default.toml` 的 `oauth_token`，有 `pages:write`）`POST /accounts/<account>/pages/projects/verto/domains {"name": "<host>"}`；再用 `cloudflared` 登入憑證 `~/.cloudflared/cert.pem`（base64 JSON：`accountID`／`zoneID`／`apiToken`）建 proxied CNAME `<host>` → `<project>.pages.dev`。cert 的 accountID 與 Pages 帳號相同。幾分鐘內網域生效、HTTPS 正常，沒有 GitHub Pages 那種憑證卡住。讀這兩個 token 需使用者明確同意，否則 classifier 會以 Credential Exploration 擋下。
- `cloudflared tunnel route dns` 只能把 hostname CNAME 到 tunnel，不能用來設 Pages 網域。
- Pages 單檔上限 25 MiB。`.unityweb` 以 `content-type: application/vnd.unity`、無 `Content-Encoding` 送出，靠 Unity decompression fallback 在 JS 解壓；Brotli + fallback 可正常載入。

## Unity 6000.4.7f1 Web build 縮小實測（Verto PR #7，合併為 `3732a6e`）

| 設定 | wasm 壓縮後 | data 壓縮後 |
|---|---|---|
| 原始：gzip、Minimal strip、OptimizeSpeed、codeOptimization BuildTimes | 14.22 MB（raw 58.7 MB） | 3.86 MB |
| 只開 High stripping | 12.24 MB | 2.92 MB |
| 只開 IL2CPP OptimizeSize | 11.70 MB | 3.85 MB |
| 只開 DiskSizeLTO | 12.88 MB（build 79 分鐘） | 3.86 MB |
| High + OptimizeSize + DiskSize（gzip） | 9.99 MB（raw 31.7 MB） | 2.91 MB |
| 再加 Brotli + 移除 visualscripting/timeline/multiplayer.center/collab-proxy | 7.56 MB | 2.27 MB |

- Web code optimization 存在 Editor 的 `UnityEditor.WebGL.UserBuildSettings.codeOptimization`（不在 git 追蹤的設定裡），所以 `Verification/BuildWeb.py` 每次 build 前設定 `WasmCodeOptimization.DiskSize`，並像檢查 splash 一樣檢查 strip／IL2CPP／compression／fallback／optimization，結果存 `TestResults/Web/size.json`，不符就失敗。
- Brotli 讓 `Web.loader.js` 從 48 KB 變 118 KB（內含 fallback decoder），但總量大幅下降。
- `Hidden/CoreSRP/CoreCopy`、`Hidden/Universal/HDRDebugView` 的 "shader is not supported on this GPU" log 在縮小前的舊 build 就有，不是 stripping 造成。
- 內建瀏覽器 pane 的 computer `key` 是瞬間按放，Unity 常收不到走路；驗證走路用 Playwright `keyboard.down/up`（`WebLoadSmoke.cjs`）。

## `WebLoadSmoke.cjs` 的檢查陷阱（Swear Review 指出並修正）

- 標題畫面被選中的圖示與斗篷同色；若按鍵被吞，往右鍵會移動選單光標，像素位移檢查會誤判通過。要像 `DemoWebSmoke.cjs` 一樣按 Enter（最多 3 次）並等 `phase-map-result` 事件，確認 level1 真的載入。
- 用「新出現像素平均 x − 消失像素平均 x」量位移是錯的：位移 d 小於 sprite 寬度 w 時結果恆等於 w，1px 也會過。改比較兩張畫面所有斗篷像素的 centroid（Verto 關卡沒有同色 UI）；實測 250 ms 往右約 88 CSS px（22 native px）。
- Phase 模板載入失敗時也會隱藏 `#loading`，原因寫在 `#error`；載入後要先斷言 `#error` 隱藏，否則壞掉的部署會晚 9 秒才以誤導訊息失敗。HTTP 404 不會觸發 Playwright `requestfailed`。
- 霓虹美術（Verto PR #9）把骨白斗篷換成粉紅兜帽＋紫色身體，`WebLoadSmoke.cjs` 改用 sprite 身體色 `3b2a78`、`261a52`、`4a3a92`、`2c2160`（`PhasePixels.Sprite` 的 c、d、b、B）找玩家；這些顏色遊戲其他地方都沒用，標題畫面也沒有。`DemoWebSmoke.cjs` 仍用舊斗篷色。

## 發布 build 的實務

- 發布用 worktree：`git worktree add --detach <path> origin/main`，`cp -cR` 主 checkout 的 `Library`，以 `open -g -j -n -a .../Unity.app --args -projectPath <path>` 背景開 Editor；完整 `BuildWeb.py`（全部 Editor／Play Mode 套件＋build）約 9 分鐘。
- Web build 會重新序列化 `Assets/DefaultVolumeProfile.asset`、`Assets/Settings/UniversalRP.asset`、`Assets/UniversalRenderPipelineGlobalSettings.asset`；一次性 checkout 裡直接還原，不要 commit。
- build 期間 `main` 可能又前進（2026-10-22 build `acab4ef` 時 PR #11 改了 favicon）；部署前再 fetch，落後就重 build 最新 `main`。
