---
title: "手動掃描要說清楚是本機命令還是 hosted workflow"
scope: tools/github
status: active
updated: 2026-09-16
evidence_digest: eb3512803133124debc5bdf3b3855afab7d94a0649e63f6474f37d1d67275913
---

# 手動掃描要說清楚是本機命令還是 hosted workflow

歷史助手交付本機掃描腳本，後承認把「手動觸發」理解成終端執行，GitHub 並沒有全庫掃描按鈕。

分開說明觸發入口、驗證範圍與實際執行狀態。腳本單元測試或 PR 自動 review 設定不代表全庫掃描已跑；此来源明確說沒有執行實際掃描且尚未提交。Hosted 路徑的認證與成本須按當次方案另核，不沿用歷史估計。
