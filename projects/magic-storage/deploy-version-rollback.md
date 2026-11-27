---
title: "Magic Storage 部署：版本與 JAR 回滾、同版本遊戲 READY"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: 3d231cb9b74e165cd7c0fa0771eee8c26cb8b0eb0e1daea71e66e3a5eb4f49d1
---

- **Build 失敗仍增加版本，或 backup 已搬走 active jars 但 rollback 清單還空。** 將建置前 bump 和產物驗證包在同一部署交易。每次搬移或替換時同步保存可復原資訊，避免 backup helper 返回、rollback list 尚未寫入就遇 BaseException。測 build fail、部分 replace、handoff 與下載前後的故障，檢查版本與完整 active tree 還原及重試不累加，並保留原始例外。回寫前核對並行修改，Python 補償未涵蓋 SIGKILL／斷電。
- **新版 build 綠但未到 READY，舊版卻能跑。** 固定同 instance 和模式比較版本，以正式 runner 啟動到 READY 驗收；focused tests、artifact hash、舊版 READY 都不能代替。關閉 fullscreen 後仍失敗只排除當次顯示模式單因，不足以證實或否定 Mixin 介面可見性。原片段僅有診斷、RED 或修正計畫，未觀察最終新版成功。

[原始紀錄](https://github.com/swear01/agent-memory/blob/9b2d556e370f3a71736d5dc0a63f3b1e1c8c3a0e/projects/magic-storage/deploy-version-rollback.md) · [JAR 搬移的故障點](https://github.com/swear01/agent-memory/blob/9b2d556e370f3a71736d5dc0a63f3b1e1c8c3a0e/projects/magic-storage/backup-rollback-registration-gap.md) · [新版啟動證據](https://github.com/swear01/agent-memory/blob/9b2d556e370f3a71736d5dc0a63f3b1e1c8c3a0e/projects/magic-storage/startup-regression-outcome.md)
