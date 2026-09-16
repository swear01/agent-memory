---
title: "Chemical isValid 與目前 tank 內容須分開"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: 2ddf307e27e81b14e0015ea9735c68923b0844dc5925aaca49b29b92b1ee56a5
---

# Chemical isValid 與目前 tank 內容須分開

保存的 review 指出 Mekanism isValid 錯把 tank 目前內容當成型別合法性；後續報告稱將不同 chemical 的拒絕移到 indexed insertion，fixture 由失敗改為通過。

按當次介面契約區分型別是否可接受與目前指定 tank 能否插入；維持 generic 多種類插入能力，不以暫時庫存內容宣告某 chemical 永遠無效。

結果是歷史 subagent 報告，fixture 僅對該次代表版本，本次未重跑。反射 linkage 診斷是另一個原因，分開記錄。
