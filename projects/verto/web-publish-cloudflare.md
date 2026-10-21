---
title: Verto Web 公開試玩：Cloudflare Pages 手動發布與 Unity build 縮小
scope: projects/verto
status: verified
updated: 2026-10-21
tags: [Verto, PHASE, Unity, WebGL, Cloudflare-Pages, wrangler, cloudflared, build-size]
---

## 為何不像 hexapharma 用 GitHub Pages 自動部署

- hexapharma 是 npm 前端，GitHub-hosted runner 就能 build。Verto 的 Unity Web build 需要有授權的 Unity 6000.4.7f1 GUI Editor + Pipeline（專案規定不得改用 `-batchmode`），GitHub-hosted runner 沒有。
- 唯一自動方案是 self-hosted Mac runner；但 public repo 掛 self-hosted runner 時，fork PR 可以帶自己的 workflow 在 Mac 上執行（repo 預設 fork 核准政策只到 `first_time_contributors`）。安裝 runner（LaunchAgent）也被 auto mode classifier 以 Unauthorized Persistence 擋下。
- 使用者選擇本地 build 後手動上傳 Cloudflare Pages，合併到 `main` 不會自動更新網站。舊的 `web.yml`（等 runner、每次 push 都失敗）已移除（Verto PR #4）。

## Cloudflare Pages 設定（2026-10-21）

- 專案 `verto`，`*.pages.dev` 名稱為 `verto-6h5.pages.dev`；自訂網域 `https://verto.5334601.xyz/`，HTTP 自動 301 到 HTTPS。
- 發布：`npx wrangler pages deploy Builds/Web --project-name verto --branch main`，再跑 `node Verification/WebLoadSmoke.cjs https://verto.5334601.xyz/`。需先 `npx wrangler login`（瀏覽器 OAuth，使用者授權）。
- Wrangler 4.151 沒有 Pages 自訂網域指令。可行做法：用 Wrangler OAuth token（`~/Library/Preferences/.wrangler/config/default.toml` 的 `oauth_token`，有 `pages:write`）`POST /accounts/<account>/pages/projects/verto/domains {"name": "<host>"}`；再用 `cloudflared` 登入憑證 `~/.cloudflared/cert.pem`（base64 JSON：`accountID`／`zoneID`／`apiToken`）建 proxied CNAME `<host>` → `<project>.pages.dev`。cert 的 accountID 與 Pages 帳號相同。幾分鐘內網域生效、HTTPS 正常，沒有 GitHub Pages 那種憑證卡住。讀這兩個 token 需使用者明確同意，否則 classifier 會以 Credential Exploration 擋下。
- `cloudflared tunnel route dns` 只能把 hostname CNAME 到 tunnel，不能用來設 Pages 網域。
- Pages 單檔上限 25 MiB。`.unityweb` 以 `content-type: application/vnd.unity`、無 `Content-Encoding` 送出，靠 Unity decompression fallback 在 JS 解壓；Brotli + fallback 可正常載入。

## Unity 6000.4.7f1 Web build 縮小實測（Verto PR #7）

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
