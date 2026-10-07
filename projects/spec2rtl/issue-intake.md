---
title: Spec2RTL issue labels and forms
scope: projects/spec2rtl
status: verified
updated: 2026-10-07
---

## 已完成的設定

- Issue 設定 PR #5 已合併至 `develop`，merge commit 為 `9373977c2bb0179c21735a8f06e5f2fb2c306bde`。
- 啟用 PR #10 只將三個 YAML 表單、chooser config 和 `CONTRIBUTING.md` 共五個檔案同步至預設分支 `main`，merge commit 為 `77d78cec89007dbdc833e5a6718b9e6a70d5e1ff`。其他 develop 功能變更沒有一起發布。
- 保留九個 GitHub 預設標籤，新增十三個，總共二十二個。初期「十二個新標籤、PR #5 待審查」的紀錄已被這次結案取代。
- 九個階段各有 `area:<skill-name>`：`orchestrator`、`refine-spec`、`plan-architecture`、`generate-rtl`、`generate-testplan`、`generate-assertions`、`lint-rtl`、`generate-testbench`、`alignment-check`。另有 `area:tooling`、`status:needs-triage`、`status:blocked`、`priority:high`。
- 三個表單為 Bug report、Feature or improvement、Usage question；檔案位於 `.github/ISSUE_TEMPLATE/`，依序命名為 `01-bug-report.yml`、`02-improvement.yml`、`03-question.yml`。`config.yml` 保留 `blank_issues_enabled: true`。表單沿用英文，CONTRIBUTING 接受英文與繁體中文回報。
- 表單自動加上問題類型與 `status:needs-triage`；dropdown 不會自動加 area label，由維護者分類。Blocked 必須說明具體阻礙；一般工作不需 priority label，也不另加 in-progress label。

## 驗證與授權界線

- YAML、必填欄位、唯一 field ID、階段名稱、live label 名稱／色碼／說明、差異範圍及遠端 main 五個檔案內容均已核對。CONTRIBUTING 已更新，兩個任務工作樹在確認合併、乾淨且遠端分支已刪除後清理。
- 使用者明確允許這次 issue 設定沒有 bot review 也可直接合併。這是本次變更的授權，不是所有專案或未來變更的永久 review 例外。此變更沒有執行 RTL regression，也沒有 bot review 通過證據。
- GitHub YAML issue forms 必須存在預設分支才可使用；只合併至 develop 不足以啟用。若 develop 已有其他功能，僅推進設定檔，避免順帶發布那些功能。
- 實際查詢中，GraphQL `Repository.issueTemplates` 對 YAML issue forms 回傳空清單，REST community profile 也未列出表單。不要據此判定設定消失；核對預設分支的 YAML 與實際 chooser。這次已核對遠端檔案，沒有以 API 清單宣稱完成瀏覽器畫面驗證。
- GitHub App 安裝與 repo 的 Bugbot 啟用狀態是不同證據；未取得對應最新 head 的 review 結果時，不可把本地檢查或 merge 當成 bot review 通過。
