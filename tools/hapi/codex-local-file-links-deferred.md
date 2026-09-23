---
title: Codex local file links 在 HAPI 有顏色但無法點擊，待實例再查
scope: tools/hapi
status: deferred
updated: 2026-09-23
---

## 使用者決定

使用者回報 Codex 回覆的 local file link 在 HAPI 顯示不同顏色，可看到目標檔案與行號，但無法點擊；與其 Codex 原生使用經驗不同。2026-09-23 暫時找不到實例，要求先記住，等下次抱怨再處理。不要主動修正、開 issue 或部署。使用者提出可能是模型自己的 bug；這不是已確認根因。

## 已確認的程式行為，並非個案診斷

檢查 HAPI checkout HEAD `2f72a8012`：

- `web/src/components/assistant-ui/markdown-text.tsx` 的 `InertMarkdownHref` 產生帶原始 href 作為 `title`、有 hint 顏色的 `span`，沒有可點擊 anchor。因此可保留檔案／行號提示卻無法點擊；尚未確認使用者那筆引用走此分支。
- `web/src/lib/markdown-href-policy.ts` 的 `classifyNoSchemeHref` 只接受 session `metadata.path` 內且副檔名在白名單的絕對路徑；工作目錄外或缺少 metadata 的目標變 inert。
- `stripLineSuffix`、fragment 處理會移除行號；`FilePathAnchor` 僅把檔案 path 與 origin 傳給預覽頁，沒有傳行號定位。
- 相關 upstream：tiann/hapi issue #1120（Markdown／inline-code 檔案引用，PR #1142 已合併）、#1452（點擊後錯誤路由，PR #1519 改為無法解析則 inert，已合併）、#1536（可點但離線 session 的檔案 RPC 失敗，調查時仍 open）。後者不同於本次描述。
- 執行 `bun run --cwd web test src/lib/markdown-href-policy.test.ts src/lib/remark-file-path-links.test.ts src/components/assistant-ui/markdown-a.test.tsx`：172 tests pass；jsdom 有不支援 navigation 的訊息。未驗證實際部署版／瀏覽器個案，未修改 HAPI 程式。

## 下次續查

先取得實際訊息原始 Markdown、完整 link target（含行號）、session 工作目錄和使用介面；若提供 HAPI session 連結，用 inspect_peer 取得上下文。檢查 DOM 是帶 title 的 inert span、href 被 sanitizer 清空的 anchor，或真正 anchor 點擊失敗。對照當時部署版本，區分模型格式錯誤、HAPI parser／policy、點擊處理與行號定位缺口；不要直接斷言是模型 bug 或工作目錄限制。
