---
title: "ICCAD2026B 資料生成：總行數、profile 停止條件與 strict ledger"
scope: "projects/ICCAD2026B"
status: active
updated: 2026-09-07
evidence_digest: 96c463e728ef26cb5b8014b5e3e3cf5a8e5c7d3483e9733ebd6e84435bde63b8
---

- **Generator validator 綠，實際資料量遠小於目標。** 核對產生後總 trace rows 與 source range，格式通過只驗形狀。用使用者真正要求的 profile exact minimum 設長跑停止條件；單 bucket 配額只是排程手段。改後續排程時保留已接受證據，勿用配額填滿宣稱 profile 達標。
- **Derived matrix 顯示完成，但 DVC promotion 未成功。** 只從正式 strict ledger 計數，隔離候選、條件性替換、失敗 promotion 各自列。修衍生報表，勿回填較大的完成數。歷史片段只有對帳及規劃修正，無新全量 generation 或 profile 達標證據。

[原始紀錄](https://github.com/swear01/agent-memory/blob/2505b5e7b1170ecc3a61858e10894b2af847b5b7/projects/ICCAD2026B/strict-ledger-counts.md) · [不足的 trace rows](https://github.com/swear01/agent-memory/blob/2505b5e7b1170ecc3a61858e10894b2af847b5b7/projects/ICCAD2026B/generated-workload-budget.md) · [profile 停止條件](https://github.com/swear01/agent-memory/blob/2505b5e7b1170ecc3a61858e10894b2af847b5b7/projects/ICCAD2026B/profile-stop-condition.md)
