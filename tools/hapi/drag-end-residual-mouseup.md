---
title: "Drag 結束後的殘留 mouseup 不應被當作 click"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: eedf1466ce596093d4acec25544c68ebeec61650fb798c6d6294815f61d079ff
---

# Drag 結束後的殘留 mouseup 不應被當作 click

助手定位 onDragEnd 呼叫 handleEnd(false)，過早把 touchMoved 清掉，使後續 mouseup 走一般 click。

讓同一個已移動手勢在終止事件處仍保持已移動判定，直到下一次真正的新手勢再初始化；分別測 click、拖動與尾隨 mouseup。

來源只有定位與修法，另有測試環境失敗，沒有修後通過證據；不把環境錯誤併成這個事件狀態根因。
