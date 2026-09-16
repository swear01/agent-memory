---
title: "Natural witness 必須由合法程式與獨立架構 oracle 區分變體"
scope: projects/ICCAD2026B
status: active
updated: 2026-09-16
evidence_digest: 603601d13d2a3013c06b6c3885d3804a47ef9458588686175258ad31274a6911
---

# Natural witness 必須由合法程式與獨立架構 oracle 區分變體

兩份保存的 subagent 報告分別指出：一個非法 opcode 在 fixed／mutant 都抑制架構副作用，差異只見於 force 觀察的內部狀態；另一個自然 JALR 候選因 buggy 也通過而被拒。

若 natural body 契約禁止 force／release，就不能拿內部 witness 充作新的自然 family；先確認合法輸入下可觀察的架構差異及獨立 oracle。這些是指定案例的調查與測試回報，沒有新增 body 或 promotion；不外推為所有可能程式都永遠不可區分。
