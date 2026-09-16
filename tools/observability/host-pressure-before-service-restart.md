---
title: "服務回應慢先核對主機資源與工作歸屬"
scope: tools/observability
status: active
updated: 2026-09-16
evidence_digest: 94807a9eef41ba75672ccce3df9c05cd8d36afdcb44b72a6dc56bad382c52c21
---

# 服務回應慢先核對主機資源與工作歸屬

歷史觀測有高 load、RAM／swap 壓力及 D-state 程序，Hub 仍 listen 並回應 200；助手卻由此承諾清掉記憶體後會恢復，並列出多個強制 kill。

分開量測服務健康、host 資源與 I/O 等待，核對實際工作擁有者、用途及 supervisor，再做有授權的限載或停止。高 RSS、跑很久或同帳號不代表可刪；D-state 也不單獨證明 NFS 根因。來源沒有處置後恢復證據，RSS 換算亦不可把 KB 直接稱 GiB。
