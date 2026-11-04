---
title: Swear Review 的 ChatGPT 額度備援與 OCR 認證隔離
scope: projects/swear-review
status: deployed on Oracle; ChatGPT authenticated; fallback disabled pending app usage limit
updated: 2026-11-04
---

- `swear01/swear-review` PR #18 已合併，merge `a42b65be7c8158ee8b71749e9a10c7569f29c98b`；最新 head `686a661` 的 CI、Gemini review 通過。包含 PR #19 session-affinity 修正的整合版 `b7bd7b8` 已部署 Oracle，原始碼與 compiled fallback 均在位，healthz／readyz 通過。部署後 `swear-review-e2e` PR #1 的真實 review 以 `deepseek-flash`／OCR 1.13.5 完成成功；這僅證明 primary review 流程。
- 實測 OCR 1.13.5 會隱藏 gateway 的 429 錯誤本文：coverage 只留下 `provider or subtask request failed`，stderr 也沒有原始 `insufficient_quota`。不能掃留言、stdout 文字或只看到 429 就切換；使用版本化 `ocr.llm-retry-report/v1`。條件是 run 為 failed/partial、未取消或逾時、coverage 有失敗且全部為 provider、至少一個 failed logical request；每個 failed request 有多次 attempt，最後為 HTTP 429／`rate_limited`／`http`，且 report 不含 cancelled 或未知 outcome。quota 耗盡與持續性限流在這份輸出中無法區分。
- DeepSeek 保持 primary；省略 `llm.chatgpt_fallback` 即關閉。符合條件才對相同 immutable SHAs 重跑一次，併發不變，兩次共用原本 hard deadline，不形成 retry loop。部分 primary findings 會保留並與備援 findings 去重；備援 skipped、覆蓋不足、失敗或逾時仍 fail closed，不能假裝完整成功。Abort/supersession 照常丟棄舊結果。GitHub summary 與 DB 記錄實際模型。
- OCR 的 ChatGPT store 固定依 HOME 找 `.opencodereview/auth/chatgpt`，包含 `accounts.json`、`host.json`、token rotation lock，沒有可用的獨立 auth-path env。不要為每個 job 複製 token 檔，會遺失 refresh rotation／造成競爭；只把隔離 HOME 的 auth 子目錄 symlink 到 service-owned persistent store，sessions 仍放 job HOME。目錄 `0700`、帳號與 host 檔 `0600`，驗證 service ownership、regular file、位於 review workspace 之外。VM 使用自己的固定 host ID；不要連同筆電 host ID 一起複製。
- `--provider chatgpt` 會優先於 primary LLM env，但仍需要隔離 HOME 內的非秘密 `config.json` 含 `providers.chatgpt: {}`。備援顯式選 provider／可選 model，並移除 gateway token、endpoint 與私有 headers。真人 OAuth、plan permission 與模型可用性須另行驗證；單元測試、mock quota capture 或合併 PR 都不能代替這個證據。
- 真人 OAuth 與 plan scopes 已驗證，受保護的帳號 store 已匯入 Oracle，保留 VM 自己的 host ID；本機 bootstrap 憑證副本已刪除。真實 native probe 在 primary 六次 HTTP 429 後確實嘗試 ChatGPT `gpt-6.1-sol`，但回傳 `subscription_sharing_usage_limit_exceeded`，沒有成功的 ChatGPT inference。此錯誤可能是應用層上限，不代表整個 ChatGPT 方案額度耗盡；官方要求暫停 plan requests，不可猜測重置時間。正式 runtime 未設定或啟用 `llm.chatgpt_fallback`，等待使用者檢查 Usage；登入成功與 fallback 成功必須分開報告。
