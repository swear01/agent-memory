---
title: NTU DVLab Discord 伺服器架構、身份組分流與頻道治理規範
scope: projects/dvlab-mis
project: dvlab-mis
tags: [discord, dvlab, permissions, roles, channels, onboarding, mis]
status: active
created: 2026-09-18
updated: 2026-09-18
---

- **把校友休閒權限和內部研究權限一起移動。** 共用公告、一般／語音、活動保留校友可用；研究、GPU 及網管依各自權限。研究題與遊戲頻道採 opt-in，屆次只讓該屆與老師看。用 Discord 原生 Community Onboarding 的身分單選、休閒多選、研究多選、屆次單選。操作後分別用在學生、校友、未領取及已領取身分讀回可見性。
- **舊 EDA 或 R11 要改區、改名。** 沿用 channel ID 並更新 name／parent_id，先核類別與 channel permission overwrite，保留舊聊天及附件。把過期公告歸到獨立受限封存區。原身分與頻道清單見下方快照，成員現況先由正式伺服器讀回。
- **角色調整報 403 或連續請求遇 429。** Bot 的最高角色須高於目標角色，再查權限。批次按 Retry-After 節流並讀回變更。若成員 API 因 intent 不足拒絕，使用有授權的名單或限定搜尋，別將搜尋結果當完整名單。

[完整原文](https://github.com/swear01/agent-memory/blob/06f50e3a42e850b37cffb6b6a4a446ef670e9625/projects/dvlab-mis/discord-server-architecture-and-governance.md)
