---
title: "有靜態資產與 API 回應仍不足以證明 Web UI 正常"
scope: tools/web
status: active
updated: 2026-09-16
evidence_digest: e87f152beec6157355d736955a196d9fbf473e0fca90bd72fedd300868053a4c
---

# 有靜態資產與 API 回應仍不足以證明 Web UI 正常

歷史助手發現 Glances 的一種安裝缺少 public 資產，另一種安裝有資產便宣稱 Web UI 可用；後續探測雖有首頁與 API 200，指定 JavaScript 及 favicon 路徑卻回 404。

核對實際服務版本、首頁引用的資產路徑及瀏覽器渲染。磁碟上有 public 不代表服務正在供應正確檔案；任意猜測的資產 URL 回 404 也不足以證明整個安裝損壞。來源沒有完整畫面驗收，不外推到所有 APT 或 pip 安裝。

正式站台要核對需求裡各條具體路徑、listener 和站台配置，再移除臨時預覽。歷史案例中 nginx active、根目錄 200，dashboard、overview 與 JSON 路徑卻全 404，未取得後續恢復證據。見 `git show 06f50e3:tools/web/target-route-before-preview-cleanup.md`。
