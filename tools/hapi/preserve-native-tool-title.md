---
title: "正規化工具事件時保留可用的原生 title"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: d06769ecd0d1c8728d3d81cb19cd0f15b1db1361d5dc7a8b94a5c2ca139e153f
---

# 正規化工具事件時保留可用的原生 title

歷史助手讀取 adapter 後回報 ACP title 被拿來推導 command／path，送入共用格式時卻未保留；OpenCode title 又只包在完成 output，前端因而缺少可用的呼叫描述。

沿原生事件、正規化與顯示層保存已提供的 title 及來源身份，再用真實參數補不足。工具定義 description 或附近 reasoning 不自動對應本次呼叫，更不能從相鄰事件猜一整組共同目的。來源是閱讀與方案，沒有修復或端到端驗證，provider 能力表不作現行通用規格。
