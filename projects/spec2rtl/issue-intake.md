---
title: Spec2RTL issue labels and forms
scope: projects/spec2rtl
status: verified
updated: 2026-10-07
---

- **YAML 已合進 develop，chooser 卻沒新表單。** GitHub Issue Forms 用預設分支。只同步 `.github/ISSUE_TEMPLATE/` 的三個 YAML、`config.yml` 與 CONTRIBUTING，避免夾帶 develop 的其他功能。GraphQL `Repository.issueTemplates` 和 REST community profile 曾漏列 YAML；要核預設分支的實際檔案及登入後的 chooser。
- **選了 area 但 issue 沒有 label。** 三個英文表單是 bug／improvement／question，接受英文與繁體中文，保留 blank issues；自動加類型和 `status:needs-triage`，area dropdown 由維護者套 `area:<skill-name>`。Blocked 要寫阻礙。單次免 bot review 是當次授權；App 安裝或有 merge 不能替代最新 head 的實際 review。

[原始紀錄](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/projects/spec2rtl/issue-intake.md)
