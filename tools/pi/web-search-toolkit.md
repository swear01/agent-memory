---
title: Pi 網路搜尋工具鏈（fleet 全機）
scope: tools/pi
status: active
confidence: high
created: 2026-08-26
updated: 2026-08-26
tags:
  - pi
  - web-search
  - fleet
---

# Pi 網路搜尋工具鏈（fleet 全機）

- status: active
- scope: tools/pi, fleet-wide
- updated: 2026-08-26

完整單一事實來源：`<project-root>/docs/web-search-toolkit-plan.md`（playground 仓庫）。本筆記只存跨機器可重用的重點。

## 現狀（2026-08-26）

- 七台（Mac + mazu/athena/cthulhu/valkyrie 共用 NFS `<remote-home>/swear01` + zeus `<remote-home>/swear02` + oracle `<remote-home>/ubuntu`）都用上游 `npm:pi-web-access@0.25.0`（360K DL/月），已移除 lean fork。footprint 實測 6,227（lean）→ 8,143（精簡後），用戶接受。
- 0.25.0 tools：`web_search` / `source_check` / `fetch_content` / `get_search_content`（`code_search` 已砍掉，code 查詢走 Exa provider；要 port 回 lean 的 code-search.ts 約 100 行）。
- 精簡設定在 `~/.pi/web-search.json.template`：`workflow:"none"`、sourceCheck/curator/google-account/image/autoOpenBrowser/allowBrowserCookies 全關；保留 PDF 抽取、GitHub clone。
- key 機制：`~/.secrets`（`export X_API_KEY=...`，chmod 600）＋ `~/.local/bin/pi-websearch-apply` 渲染 `~/.pi/web-search.json`。只把**非空** key 寫成 credential 引用；改 key 後每台跑一次 apply 即可，呼叫時才 source，不用重啟 pi。
- 已生效 8 把 key：Exa / Brave / Tavily / Bocha / Serper / SerpBase / Parallel / Firecrawl（Perplexity 暫空：無 free tier、須綁卡）。
- fleet 同步要點：NFS 四台寫 mazu 一次；zeus/oracle 各一次。`~/.secrets` 合併時若遠端已有空值佔位行，append-only 會跳過——要「刪舊行＋貼新值」。

## 已驗證的坑（全部實測過）

1. **credential command 內層必須用單引號**。`!sh -c ". \"$HOME/.secrets\" 2>/dev/null; printf \"%s\" \"${X:-}\""` 這種雙引號寫法會讓外層 shell 搶先展開 `${X}`（outer 環境是空的）→ 所有 key 解析為空。正確：`!sh -c '. "$HOME/.secrets" 2>/dev/null; printf "%s" "${X:-}"'`。
2. **pi-web-access 對空 credential 硬錯誤且不回退**：空引用會丟 `command-empty` / `environment-empty`，且**不會**回退到 keyless Exa MCP——所以未登記的 key 必須整組省略（apply script 過濾），不能留空值。
3. **Firecrawl 0.25.0 沒有預設 base URL**：必須在 config 設 `firecrawlBaseUrl: "https://api.firecrawl.dev"`（已進 template）。
4. **Parallel**：explicit-only（不進 auto/all 鏈）。免費額度（註冊最高 $80 + 每月 $5）需要**綁信用卡**才自動生效；402 = 帳戶 0 credits。`parallel-mcp` 若設了 key 也會拿 key 認證，不會自動落到匿名免費層。key 全 fleet 共用同一帳戶。
5. **dash 不支援 `${!k}`**：apply script 的 key 值提取必須走 `bash -c`。

## Provider 事實（2026-08 查證）

- **Baidu**：官方 API 是百度雲千帆 AppBuilder（`POST https://qianfan.baidubce.com/v2/ai_search/web_search`，header `X-Appbuilder-Authorization: Bearer <AppBuilder API Key>`，建議變數 `BAIDU_APPBUILDER_KEY`），每日 100 次免費；pi-web-access **沒有** baidu provider → P2 寫小 adapter 或走 SearXNG。
- **You.com**：免 key free MCP `https://api.you.com/mcp?profile=free`（100 次/天）；pi-web-access 沒有此 provider → P2 可選 adapter。
- 不用補：Google CSE（不開放新客戶、2027-01-01 下線）、Bing Search API（2025-08-11 退役）、Ecosia（Bing reseller）、Naver/360/Qwant。
- 可選自架：SearXNG（Docker，`?format=json`，一接 272 引擎含 baidu/yandex/marginalia/mojeek）→ config 加 `searxngBaseUrl`。
- Common Crawl Index：免 key，`curl 'https://index.commoncrawl.org/CC-MAIN-XXXX-XX-index?url=...&output=json'`（CDXJ）查 URL 索引。

## 未決事項

- mazu 的 Codex OAuth `refresh_token_reused`：**已修復**（2026-08-26 實測 `pi auth check --provider openai-codex` = ready；登入流程見 `tools/pi/openai-codex-login.md`）。
- P2：Baidu adapter、You.com adapter（都要才做）。
