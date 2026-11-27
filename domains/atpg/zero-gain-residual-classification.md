---
title: "ATPG 零新增偵測不等於所有 residual 都不可測"
scope: "domains/atpg"
status: active
updated: 2026-09-09
evidence_digest: 39068903ad7fc5f299fe86f899d48ed415297c69d3200a2a8d09d9b9d2032cb4
---

# ATPG 零新增偵測不等於所有 residual 都不可測

- Progressive residual 在 T2／T4 新增偵測為零，保留該設計及設定下的結果。對照 R1_count、T1_AU、AB、TO 和其他狀態，區分「演算法未找到」與「structurally untestable」。既有 CSV 已見 R1_count 大於 T1_AU，不能將餘數全部歸成 AU，也不能從一次零增益推更長 frame 永遠無效。 [原始 CSV 對照與可測性限制](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/domains/atpg/zero-gain-residual-classification.md)

## 零增益與不收斂的診斷

- `storeCurrentAtpgVal` 反覆報 `numAssignedValueChanged != 0` 時，即使 exit 0 與 coverage 正常輸出，也要先保存 invariant failure 並重現狀態更新／回溯。大量 AU 不能代替回溯導致 misclassification 的因果證據。[不變式錯誤與 coverage 證據](https://github.com/swear01/agent-memory/blob/9b2d556e370f3a71736d5dc0a63f3b1e1c8c3a0e/domains/atpg/invariant-failure-before-coverage-cause.md)
- Exit 124 且迭代計數增加但 gate 和 backtrack 無變化時，逐一查 `findFinalObjective` 無 objective 的 caller 是否終止或真正回溯。不要以計數成長或只把 loop 推到較高層判定修好。[無 objective 的 timeout 證據](https://github.com/swear01/agent-memory/blob/9b2d556e370f3a71736d5dc0a63f3b1e1c8c3a0e/domains/atpg/no-objective-backtrack-path.md)
- 平行 progressive residual 驗各 worker 自己的 `Atpg` 物件已收到必要設定。Master 的 `nonscanDisconnectInfo` 有值不證明 worker 取得同一份設定。用相同輸入和配置做 before/after，沒有修正後重跑不得把這個假說當 T2/T4 零增益根因。[worker 傳播假說與尚未重跑的證據](https://github.com/swear01/agent-memory/blob/9b2d556e370f3a71736d5dc0a63f3b1e1c8c3a0e/domains/atpg/worker-local-configuration.md)

- 工具 blocker 不會自動改變研究問題。Sequential ATPG 原範圍可修工具時先修並重跑；自製 simulator、新演算法或改用不同實驗方向，另依使用者授權。[原研究範圍更正](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/domains/atpg/tool-blocker-research-scope.md)
