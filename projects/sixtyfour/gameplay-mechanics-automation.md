---
title: Sixty Four 後期數值公式、泯滅保護與自動化機制
scope: project
project: sixtyfour
status: active
updated: 2026-10-31
---

- **深層不再產黑／紫，主坑突然失去破碎加成。** 後期原礦偏 Celestial Foam 與 Beta-Pylene，先以 `converter41` 黃轉黑（8,192→16,384），`converter13` 黑轉紫（4,096→256），保留紫給 Injector。白 Elmerine 耗盡會讓 Silo 和 Destabilizer 一起停；用 `converter32` 紫轉白，避免同時擴張抽光黑黃。實測穩態起點 4／4／3，按儲備節流而非照舊快照照抄。
- **轉化器在料倉旁還要手動裝料，或自動裝料消耗超預期。** Native Silo 支援九宮格內相鄰可 refill 機器，普通 16 次、MKII 64 次；先核實相鄰、燃料及玩家庫存。範圍外才由 keeper 發原生操作。Hell Vault 每座保護 1,024 Hell Gem；到達深層後，超出容量的部分會被泯滅。只看總產率會忽略保護量。
- **鉻岩（Chromalit）長期過不了造價。** 高速下它會衰變，`converter76` 消耗 8,192 Foam 產 2,048 Chromalit 和 1 Hell Gem；保留 Foam 給 Chasm，再在原生生產形成短峰時購買。永久庫 `vessel2` 容量 32,768，第一座可利用原生拆解已不需的建物所返還鉻岩。前後讀原生餘額和衰變；減速後再做終局購入，別把封在容器內的鉻岩算可支付。
- **拿到 Void Sculpture，仍找不到回程或 Reality 生產。** 原生順序 Strange2→Void Sculpture→Waypoint→Reality→Strange3→Chasm→Gradient Well→Pinhole。互動 Waypoint 消耗 512 Hell Gem 進暗面，暗面 Reality 達 512 後點雕塑返回並啟用雙向，先核實所在 plane。種花成本指數成長，別耗掉 Chasm 需要的 Void 與 Hollow Stone。
- **Gradient Well 點擊不動或產出減半。** 平地建置有 penalty 0.5；暗面先造 Puncture 再蓋井消除。井由相鄰 Entropic 系列敲擊，只有 `destabilizer`／`destabilizer2` 放大，`destabilizer2a` 不適用。以 Conductor 把 Chasm 和 Silo／井接通，核對有原生 tap 且入帳。
- **Pinhole 放置 409，看似材料都夠。** 降速、用原生回收釋放 `vessel2` 儲備後在 Chasm 上升級；驗 `stats.gameIsFinished`、ENDBEGINNING 和原生存檔，遊戲完成與個別 Steam 成就另核對。已建 excavator 的存檔不能取 NEOPHOBIA；曾以直接建物診斷其他成就，其結果和原生付費操作分開。

[完整原文](https://github.com/swear01/agent-memory/blob/06f50e3a42e850b37cffb6b6a4a446ef670e9625/projects/sixtyfour/gameplay-mechanics-automation.md)
