---
title: "不需要 stacks 的 resolver 仍可能執行有效性檢查"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: c0ca2c5ea3485e60884b3aa7a6b3ed73533b77e667ffde037d8d9e588564c39b
---

# 不需要 stacks 的 resolver 仍可能執行有效性檢查

保存的 read-only audit 指出 catalog 因 adapter 不需要 available stacks 就直接回傳 baseMatch，略過 built-in 與 legacy family 對 null level 或空 output 的拒收；下游雖再次過濾，resolver 結果與記錄的 variant 數已不等價。

快速路徑必須滿足原 resolver 的完整前置契約，不能只憑 stack independence 推論身份已解析。來源建議只沿用已完成 typed plan，並回報對應 static test 失敗；沒有修後 runtime matrix，不能因下游保護存在便称此最佳化安全。
