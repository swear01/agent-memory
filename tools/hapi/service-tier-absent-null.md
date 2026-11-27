---
title: Codex Fast 與 Standard：HAPI service tier 的三態與讀回
scope: tools/hapi
status: needs-review
updated: 2026-11-28
---

- Fast／Standard 是 service tier，與 reasoning effort 分開。顯示選項前查使用中的型號、app-server catalog、登入身份與有效能力。不能由 picker 成功或靜態 catalog 判定實際模型請求已生效。核對目標版本的真實 request/schema，非法值應明確拒絕。
- 先保留未設定（`undefined`）、明確清除（`null`）及具體值的契約。歷史問題是 bootstrap 將 `currentServiceTier` 缺值轉成 `null` 然後在 keepalive 持久化，可能在操作前覆蓋 Fast 或帳號預設；核對 bootstrap、setter、同步、持久化及 resume，不直接繼承另一條路徑的預設。
- 歷史 mock 缺 `sessionInfo`，有待修測試，不能將舊 CI 或測試提案當成部署證據。以與公開 build／認證相同的實際 runtime 檢查 wire 值及重新連線後的有效設定，再聲稱 provider-dependent availability 通過。Reserve 額度是另一條狀態，見 [Luna Reserve](codex-luna-reserve.md)。

[原始 Fast 概念與限制](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/hapi/codex-fast-service-tier.md) · [原始 bootstrap 案例](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/hapi/service-tier-absent-null.md)

新增 required 欄位時列出 production API 的 Session constructors、fixtures、schema 和回讀消費者。歷史 `serviceTier` typecheck 失敗曾被誤判成重複套件，實際是 production literal 和 fixtures 同時缺欄位；不要只改測試。該案例沒有最後整套驗證。[建構處原案](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/hapi/required-session-field-all-constructors.md)
