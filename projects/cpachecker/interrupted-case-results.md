---
title: "中斷批次應保留可驗證的已完成案例"
scope: "projects/cpachecker"
status: active
updated: 2026-09-28
evidence_digest: 32ed625b90bc74bae3b0c7fcae42199fabd31c67ceaf7f967f64ffffbfa22aeb
negative_result: false
redaction: passed
---

# 中斷批次保留逐案例的終局證據

- HAPI polling `aborted`／`Unknown process id` 時核對主機啟動時間、前後 `boot_id`、launcher/owned descendants 和 terminal sidecar。固定身分使用穩定欄位，勿用可因開機改變的 `MemTotal`。新 boot 的 unit inactive/not-found 或 claim 可取不證明舊 boot 已完成。缺少 launcher exit code 保持 unknown。
- 盤點零 solver attempt 的 wrapper setup failure、可獨立驗證的 terminal、已開始但無終局的 interrupted、未開始。不完整 XML 只有在 protocol 允許逐案例採納且來源、完整與終局可驗證時保留 finished case。接續前核 runtime/JDK/source/config/資源上限，跨主機或重開機補跑不得算乾淨 paired timing。
- 有授權的 supplemental attempt 用新輸出路徑及遞增編號，不覆蓋 partial/hash。未開始案例從 attempt 1 起。分開 solver attempts、terminal slots、wrapper attempts；不接續 provider 安全 halt 或覆蓋整批排除協議。
- 工作者 deadline 不等於 whole-service wall。採用略早 child deadline 留收尾空間，仍檢查最終 hard cap、超限留 INVALID；#296 的 900 秒契約後以 895 秒截止 child 起試，5 秒不是普遍安全界。
- 保存回答按 request/response hash 對齊，能計 token 下界；沒保存的 inflight 不是零成本。不要把完成 call latency 加總當全程等待，更換 runtime 後的批次不能配為原來的效果對照。

[原始紀錄](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/projects/cpachecker/interrupted-case-results.md)

合併自 [MemTotal 與主機身分](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/projects/cpachecker/reboot-memory-identity.md)。
