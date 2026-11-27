---
title: GUPS 課程帳號：FreeIPA 權限、SSH 入口與寄送驗證
scope: projects/ntu-gups
status: verified
updated: 2026-11-04
---

- **新帳號可登入卻讀不到 PDK 或教材。** 用 Drive GUPS 助教資料的 `gups_accounts.sh`，輸入每行「帳號 Email 姓名」，允許姓名含空白。FreeIPA 加 `pdk_approved`、在 ieda1 建 0700 home。數字開頭帳號實測可用，別自造前綴。以該帳號核 `id` 與教材／PDK 可讀，實際複製 tutorial；個別名單留在私人管理來源。
- **寄件成功卻找不到寄件備份，或提供錯的登入主機。** 本流程的 `himalaya -a ntu --no-save` 不存寄件備份。英文信先核 `--preview`（只顯示首封），需重寄走既有 `--reset` 改密碼流程，不印、不存密碼。學生登入用 `ssh <account>@intl.gups.ntu.edu.tw`；當時 ieda2 尚未設定，不建議任選主機。另見 [Windows GUI 指南](windows-x11-eda-guide-site.md) 與 [期末教材](final-project-package.md)。
