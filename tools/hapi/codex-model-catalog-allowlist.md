---
title: HAPI Codex model restriction depends on an explicit local catalog
scope: tools/hapi
project: hapi
tool: Codex app-server
status: active
confidence: high
created: 2026-09-03
updated: 2026-11-28
tags:
  - hapi
  - codex
  - model-catalog
  - allowlist
source_refs:
  - hapi:canonical-gist
  - local:<project-root>/hapi-handover.md
  - live:zeus-2026-09-03
  - live:swop-2026-09-04
  - github:swear01/transfer_MAC#38
generated_by: openai-codex/gpt-5.6-sol
---

- **改了預設模型，舊模型仍顯示。** HAPI 新 session picker 讀 Codex app-server 的 `model/list`；`model = "..."` 只改預設。每個獨立 home 同時需要絕對 `model_catalog_json` 和有效 `models.allowlist.json`。缺任一項時 client 曾 fail open 到內建 catalog。
- **更新 JSON 還看不到新世代。** 查同帳號、同 client 的 live catalog 和 `minimal_client_version`。2026-11-19 對照，0.165.0 無 6.2，0.170.0 有，entry 最低值 0.166.0。桌面 app 自帶另一版 app-server。更新後重開 app-server，再查 `model/list` 或 HAPI machine API；舊程序會保留啟動時快照。存在於 catalog 和真實推論要分開寫。
- 使用 transfer_MAC 的 canonical `render-codex-models`。共用 NFS home 只寫一次，各 host 各自讀回；既有 session 保留自己的模型。Windows 曾只修 live 檔卻在 restore 後缺 updater，因此 renderer 要同時驗 updater、catalog 結構、可見 slugs 和絕對路徑。缺依賴或無效 catalog 應停下。
- 保留 Astra、Sol、Terra、Luna 四階最新一代，hidden `codex-auto-review` 例外；使用者已要求移除 Daybreak，updater／restore 不加回。以 generator policy 和 live 有效集合驗證，別用 JSON 筆數。2026-11-19 的快照是 6.2 Astra／Sol、6.1 Terra、6 Luna，預設 6.2 Sol。下次查當時 catalog。
- Canonical generator 改動見 [shared-skills #67](https://github.com/swear01/shared-skills/pull/67) 和 [transfer_MAC #94](https://github.com/swear01/transfer_MAC/pull/94)。HAPI 手冊鏡像依 [canonical gist 規則](canonical-gist-sync.md) 核對。

[整理前完整證據](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/tools/hapi/codex-model-catalog-allowlist.md)
