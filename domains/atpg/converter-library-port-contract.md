---
title: "Netlist 轉換器必須核對實際 library port"
scope: "domains/atpg"
status: active
updated: 2026-09-15
evidence_digest: 0c67bbf89578b1449c55076e6f593adbfd70d6307aec18507b67782408c9e3e4
---

# Netlist 轉換器必須核對實際 library port

歷史來源保留了 FAN 的 port mismatch 錯誤與實際 library 宣告。Agent 先把錯誤訊息中的空格當成根因；隨後 library 片段顯示，該次載入的 AND2_X1 宣告輸出為 ZN，而轉換器使用 Z，Agent 承認轉換器假設錯誤。

遇到 cell port 不匹配，先逐項核對產出的 instance、當次載入的 library 與 port 宣告，再改轉換器。不要把 diagnostic 的排版或熟悉的 cell 名稱當作介面契約；同名 cell 的 port 必須以實際 library 為準。

來源支持這次 port 不一致與錯誤歸因，不證明所有 NanGate library 都採同一命名，也不證明後續轉換或 ATPG 已成功。
